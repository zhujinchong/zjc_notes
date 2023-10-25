本文主要是用于POI解析大文件Excel容易出现内存溢出的现象而提出解决方案，故此解决了大数据量的Excel文件解析的难度，在此拿出来贡献给大家，谢谢！ 

# 1. Excel2003与Excel2007
2003版最大行数是65536行，最大列数是256列。
2007版及以后的版本最大行数是1048576行，最大列数是16384列。
excel2003是以二进制的方式存储，这种格式不易被其他软件读取使用；而excel2007采用了基于XML的ooxml开放文档标准，ooxml使用XML和ZIP技术结合进行文件存储，XML是一个基于文本的格式，而且ZIP容器支持内容的压缩，所以其一大优势是可以大大减小文件的尺寸。

# 2. POI解析Excel
Apache POI是Apache软件基金会的开放源码函式库，POI提供API给Java程序对Microsoft Office格式档案读和写的功能。

POI**读取**Excel有两种模式，一种是用户模式，一种是SAX事件驱动模式，将xlsx格式的文档转换成CSV格式后进行读取。用户模式API接口丰富，使用POI的API可以很容易读取Excel，但用户模式消耗的内存很大，当遇到很大sheet、大数据网格，假空行、公式等问题时，很容易导致内存溢出。POI官方推荐解决内存溢出的方式使用CVS格式解析，即SAX事件驱动模式。

用户模式提供的方法：

* HSSFWorkbook － 提供读写Microsoft Excel格式档案的功能。(2003版本，xls)
* XSSFWorkbook － 提供读写Microsoft Excel OOXML格式档案的功能。(2007版本，xlsx)

如果是大数据量，上面HSSF和XSSF容易内存溢出。所以POI3.8提供了SXSSFWorkbook类，采用缓存方式进行大批量写文件。SAX事件驱动模式读取大文件。

# 3. POI版本

POI3.17版本和POI3.9版本差别巨大，因为项目已才用3.9版本，不好升级，所以解决POI3.9版本大文件读的问题。

**如果方便请采用POI3.17版本，另外可以使用Hutools或者Alibaba EasyExcel工具读写大文件，他们都是基于POI3.17，调用更方便。**

```
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>3.9</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>3.9</version>
</dependency>
<!--有xercesImpl才可以用sax-->
<dependency>
    <groupId>xerces</groupId>
    <artifactId>xercesImpl</artifactId>
    <version>2.9.1</version>
</dependency>
```

# 4. 代码

注意：

* 2007代码只支持字符串和数字。
* 2003代码虽然有公式，但是没有进行测试。
* 2003代码会遍历所有sheet；2007代码可以仅仅读取指定sheet。

## 1. 业务逻辑接口

```
public interface ExcelRowReaderInterface {
    void getRows(int sheetIndex, int curRow, List<String> rowlist);
}
```

## 2. 读Excel2003

```
import org.apache.poi.hssf.eventusermodel.EventWorkbookBuilder.SheetRecordCollectingListener;
import org.apache.poi.hssf.eventusermodel.FormatTrackingHSSFListener;
import org.apache.poi.hssf.eventusermodel.HSSFEventFactory;
import org.apache.poi.hssf.eventusermodel.HSSFListener;
import org.apache.poi.hssf.eventusermodel.HSSFRequest;
import org.apache.poi.hssf.eventusermodel.MissingRecordAwareHSSFListener;
import org.apache.poi.hssf.eventusermodel.dummyrecord.LastCellOfRowDummyRecord;
import org.apache.poi.hssf.eventusermodel.dummyrecord.MissingCellDummyRecord;
import org.apache.poi.hssf.model.HSSFFormulaParser;
import org.apache.poi.hssf.record.BOFRecord;
import org.apache.poi.hssf.record.BlankRecord;
import org.apache.poi.hssf.record.BoolErrRecord;
import org.apache.poi.hssf.record.BoundSheetRecord;
import org.apache.poi.hssf.record.FormulaRecord;
import org.apache.poi.hssf.record.LabelRecord;
import org.apache.poi.hssf.record.LabelSSTRecord;
import org.apache.poi.hssf.record.NumberRecord;
import org.apache.poi.hssf.record.Record;
import org.apache.poi.hssf.record.SSTRecord;
import org.apache.poi.hssf.record.StringRecord;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.poifs.filesystem.POIFSFileSystem;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

public class ExcelReader03EventModel implements HSSFListener {
    private int minColumns = -1;
    private int maxColNum;   // 第一行确定maxColNum

    private POIFSFileSystem fs;

    private int lastRowNumber;

    private int lastColumnNumber;

    /**
     * Should we output the formula, or the value it has?
     */
    private boolean outputFormulaValues = true;

    /**
     * For parsing Formulas
     */
    private SheetRecordCollectingListener workbookBuildingListener;

    // excel2003工作薄
    private HSSFWorkbook stubWorkbook;

    // Records we pick up as we process
    private SSTRecord sstRecord;

    private FormatTrackingHSSFListener formatListener;

    // 表索引
    private int sheetIndex = -1;

    private int onSheet = -1;

    private BoundSheetRecord[] orderedBSRs;

    @SuppressWarnings("unchecked")
    private ArrayList boundSheetRecords = new ArrayList();

    // For handling formulas with string results
    private int nextRow;

    private int nextColumn;

    private boolean outputNextStringRecord;

    // 当前行
    private int curRow = 0;

    // 存储行记录的容器
    private List<String> rowlist = new ArrayList<String>();

    @SuppressWarnings("unused")
    private String sheetName;

    private ExcelRowReaderInterface rowReader;

    public ExcelReader03EventModel(ExcelRowReaderInterface rowReader) {
        this.rowReader = rowReader;
    }

    /**
     * 遍历excel下所有的sheet
     */
    public void processOneSheet(File file, int onSheet) throws IOException {
        this.onSheet = onSheet;
        this.fs = new POIFSFileSystem(new FileInputStream(file));
        MissingRecordAwareHSSFListener listener = new MissingRecordAwareHSSFListener(this);
        formatListener = new FormatTrackingHSSFListener(listener);
        HSSFEventFactory factory = new HSSFEventFactory();

        HSSFRequest request = new HSSFRequest();
        if (outputFormulaValues) {
            request.addListenerForAllRecords(formatListener);
        } else {
            workbookBuildingListener = new SheetRecordCollectingListener(formatListener);
            request.addListenerForAllRecords(workbookBuildingListener);
        }
        factory.processWorkbookEvents(request, fs);
    }

    public void processAllSheet(File file) throws IOException {
        this.fs = new POIFSFileSystem(new FileInputStream(file));
        MissingRecordAwareHSSFListener listener = new MissingRecordAwareHSSFListener(this);
        formatListener = new FormatTrackingHSSFListener(listener);
        HSSFEventFactory factory = new HSSFEventFactory();

        HSSFRequest request = new HSSFRequest();
        if (outputFormulaValues) {
            request.addListenerForAllRecords(formatListener);
        } else {
            workbookBuildingListener = new SheetRecordCollectingListener(formatListener);
            request.addListenerForAllRecords(workbookBuildingListener);
        }
        factory.processWorkbookEvents(request, fs);
    }
     /**
     * HSSFListener 监听方法，处理 Record
     */
    @Override
    @SuppressWarnings("unchecked")
    public void processRecord(Record record) {
        // 此处是我提取出来的，可以和下面switch代码合并。
        // 提取的原因是读取指定sheet，否则代码会遍历所有sheet
        switch (record.getSid()) {
            case BoundSheetRecord.sid:
                boundSheetRecords.add(record);
                break;
            case SSTRecord.sid:
                sstRecord = (SSTRecord) record;
                break;
            case BOFRecord.sid:
                BOFRecord br = (BOFRecord) record;
                if (br.getType() == BOFRecord.TYPE_WORKSHEET) {
                    // 如果有需要，则建立子工作薄
                    if (workbookBuildingListener != null && stubWorkbook == null) {
                        stubWorkbook = workbookBuildingListener.getStubHSSFWorkbook();
                    }

                    sheetIndex++;
                    if (orderedBSRs == null) {
                        orderedBSRs = BoundSheetRecord.orderByBofPosition(boundSheetRecords);
                    }
                    sheetName = orderedBSRs[sheetIndex].getSheetname();
                }
                break;
            default:
                break;
        }
        if (this.onSheet != -1 && this.sheetIndex > this.onSheet) {
            return;
        }
        if (this.onSheet == -1 || this.sheetIndex == this.onSheet) {
            int thisRow = -1;
            int thisColumn = -1;
            String thisStr = null;
            String value = null;
            switch (record.getSid()) {
                case BlankRecord.sid:
                    BlankRecord brec = (BlankRecord) record;
                    thisRow = brec.getRow();
                    thisColumn = brec.getColumn();
                    thisStr = "";
                    rowlist.add(thisColumn, thisStr);
                    break;
                case BoolErrRecord.sid: // 单元格为布尔类型
                    BoolErrRecord berec = (BoolErrRecord) record;
                    thisRow = berec.getRow();
                    thisColumn = berec.getColumn();
                    thisStr = berec.getBooleanValue() + "";
                    rowlist.add(thisColumn, thisStr);
                    break;

                case FormulaRecord.sid: // 单元格为公式类型
                    FormulaRecord frec = (FormulaRecord) record;
                    thisRow = frec.getRow();
                    thisColumn = frec.getColumn();
                    if (outputFormulaValues) {
                        if (Double.isNaN(frec.getValue())) {
                            // Formula result is a string
                            // This is stored in the next record
                            outputNextStringRecord = true;
                            nextRow = frec.getRow();
                            nextColumn = frec.getColumn();
                        } else {
                            thisStr = formatListener.formatNumberDateCell(frec);
                        }
                    } else {
                        thisStr = '"' + HSSFFormulaParser.toFormulaString(stubWorkbook, frec.getParsedExpression()) + '"';
                    }
                    rowlist.add(thisColumn, thisStr);
                    break;
                case StringRecord.sid:// 单元格中公式的字符串
                    if (outputNextStringRecord) {
                        // String for formula
                        StringRecord srec = (StringRecord) record;
                        thisStr = srec.getString();
                        thisRow = nextRow;
                        thisColumn = nextColumn;
                        outputNextStringRecord = false;
                    }
                    break;
                case LabelRecord.sid:
                    LabelRecord lrec = (LabelRecord) record;
                    curRow = thisRow = lrec.getRow();
                    thisColumn = lrec.getColumn();
                    value = lrec.getValue().trim();
                    value = value.equals("") ? " " : value;
                    this.rowlist.add(thisColumn, value);
                    break;
                case LabelSSTRecord.sid: // 单元格为字符串类型
                    LabelSSTRecord lsrec = (LabelSSTRecord) record;
                    curRow = thisRow = lsrec.getRow();
                    thisColumn = lsrec.getColumn();
                    if (sstRecord == null) {
                        rowlist.add(thisColumn, " ");
                    } else {
                        value = sstRecord.getString(lsrec.getSSTIndex()).toString().trim();
                        value = value.equals("") ? " " : value;
                        rowlist.add(thisColumn, value);
                    }
                    break;
                   case NumberRecord.sid: // 单元格为数字类型
                    NumberRecord numrec = (NumberRecord) record;
                    curRow = thisRow = numrec.getRow();
                    thisColumn = numrec.getColumn();
                    value = formatListener.formatNumberDateCell(numrec).trim();
                    value = value.equals("") ? " " : value;
                    // 向容器加入列值
                    rowlist.add(thisColumn, value);
                    break;
                default:
                    break;
            }

            // 遇到新行的操作
            if (thisRow != -1 && thisRow != lastRowNumber) {
                lastColumnNumber = -1;
            }

            // 空值的操作
            if (record instanceof MissingCellDummyRecord) {
                MissingCellDummyRecord mc = (MissingCellDummyRecord) record;
                curRow = thisRow = mc.getRow();
                thisColumn = mc.getColumn();
                rowlist.add(thisColumn, " ");
            }

            // 取最大行
            if (thisRow == 0) {
                maxColNum = thisColumn + 1;
            }
            // 更新行和列的值
            if (thisRow > -1) {
                lastRowNumber = thisRow;
            }
            if (thisColumn > -1) {
                lastColumnNumber = thisColumn;
            }

            // 行结束时的操作
            if (record instanceof LastCellOfRowDummyRecord) {
                if (minColumns > 0) {
                    // 列值重新置空
                    if (lastColumnNumber == -1) {
                        lastColumnNumber = 0;
                    }
                }
                lastColumnNumber = -1;


                // 每行结束时， 调用getRows() 方法
                if (rowlist.size() < maxColNum) {
                    for (int i = rowlist.size(); i < maxColNum; i++) {
                        rowlist.add("");
                    }
                }
                List<String> curList = new ArrayList<>(rowlist.size());
                for (String s : rowlist) {
                    curList.add(s);
                }
                rowReader.getRows(sheetIndex, curRow, curList);
                // 清空容器
                rowlist.clear();

            }
        }

    }

    public static void main(String[] args) {
        ExcelRowReaderInterface rowReader = new ExcelRowReaderImpl();
        try {
            ExcelReader03EventModel excelReader03EventModel = new ExcelReader03EventModel(rowReader);
            excelReader03EventModel.processAllSheet(new File("D://test//BTS Information.xls"));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```



## 3. 读Excel2007

```
import org.apache.poi.openxml4j.exceptions.OpenXML4JException;
import org.apache.poi.openxml4j.opc.OPCPackage;
import org.apache.poi.xssf.eventusermodel.XSSFReader;
import org.apache.poi.xssf.model.SharedStringsTable;
import org.apache.poi.xssf.usermodel.XSSFRichTextString;
import org.xml.sax.*;
import org.xml.sax.helpers.DefaultHandler;
import org.xml.sax.helpers.XMLReaderFactory;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.regex.Pattern;


public class ExcelReader07EventModel {
    private int currentRow = -1;
    private int sheetIndex = -1;
    private int maxColNum;
    private List<IndexValue> rowData;
    private ExcelRowReaderInterface rowReader;

    public ExcelReader07EventModel(ExcelRowReaderInterface rowReader) throws Exception {
        this.rowReader = rowReader;
    }

    // onSheet从1开始
    public void processOneSheet(File file, int onSheet) throws Exception {
        OPCPackage pkg = OPCPackage.open(file);
        XSSFReader r = new XSSFReader(pkg);
        SharedStringsTable sst = r.getSharedStringsTable();
        XMLReader parser = fetchSheetParser(sst);
        sheetIndex = onSheet;
        InputStream sheet1 = r.getSheet("rId" + (onSheet + 1));
        InputSource sheetSource = new InputSource(sheet1);
        parser.parse(sheetSource);
        sheet1.close();
    }

    public void processAllSheet(File file) throws IOException, OpenXML4JException, SAXException {
        OPCPackage pkg = OPCPackage.open(file);
        XSSFReader xssfReader = new XSSFReader(pkg);
        SharedStringsTable sst = xssfReader.getSharedStringsTable();
        XMLReader parser = this.fetchSheetParser(sst);
        Iterator<InputStream> sheets = xssfReader.getSheetsData();
        while (sheets.hasNext()) {
            sheetIndex++;
            InputStream sheet = sheets.next();
            InputSource sheetSource = new InputSource(sheet);
            parser.parse(sheetSource);
            sheet.close();
        }
    }

    private XMLReader fetchSheetParser(SharedStringsTable sst) throws SAXException {
        XMLReader parser = XMLReaderFactory.createXMLReader("org.apache.xerces.parsers.SAXParser");
        ContentHandler handler = new PagingHandler(sst);
        parser.setContentHandler(handler);
        return parser;
    }

    private class PagingHandler extends DefaultHandler {
        private SharedStringsTable sst;
        private String lastContents;
        private boolean nextIsString;
        private String index = null;

        private PagingHandler(SharedStringsTable sst) {
            this.sst = sst;
        }

        /**
         * 每个单元格开始时的处理
         */
        @Override
        public void startElement(String uri, String localName, String name, Attributes attributes) throws SAXException {
            // c => cell
            if (name.equals("c")) {
                index = attributes.getValue("r");
                // System.out.println(index);
                //这是一个新行
                if (Pattern.compile("^A[0-9]+$").matcher(index).find()) {
                    //存储上一行数据
                    if (rowData != null && !rowData.isEmpty()) {
                        try {
                            List<String> myDataList = getMyDataList();
                            rowReader.getRows(sheetIndex, currentRow, myDataList);
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                    rowData = new ArrayList<IndexValue>();//新行要先清除上一行的数据
                    currentRow++;//当前行+1
                }
                // Figure out if the value is an index in the SST
                String cellType = attributes.getValue("t");
                if (cellType != null && cellType.equals("s")) {
                    nextIsString = true;
                } else {
                    nextIsString = false;
                }
            }
            // Clear contents cache
            lastContents = "";
        }
                //每个单元格结束时的处理
        @Override
        public void endElement(String uri, String localName, String name) throws SAXException {
            if (nextIsString) {
                int idx = Integer.parseInt(lastContents);
                lastContents = new XSSFRichTextString(sst.getEntryAt(idx)).toString();
                nextIsString = false;
            }
            if (name.equals("v")) {
                // System.out.println(lastContents);
                rowData.add(new IndexValue(index, lastContents));
            }
        }

        @Override
        public void characters(char[] ch, int start, int length) throws SAXException {
            lastContents += new String(ch, start, length);
        }

        @Override
        public void endDocument() throws SAXException {
            if (rowData != null && !rowData.isEmpty()) {
                try {
                    List<String> myDataList = getMyDataList();
                    rowReader.getRows(sheetIndex, currentRow, myDataList);
                } catch (Exception e) {
                    e.printStackTrace();
                }

            }
        }
    }

    private class IndexValue {
        String v_index;
        String v_value;

        public IndexValue(String v_index, String v_value) {
            super();
            this.v_index = v_index;
            this.v_value = v_value;
        }

        @Override
        public String toString() {
            return "IndexValue [v_index=" + v_index + ", v_value=" + v_value + "]";
        }

        public int getLevel(IndexValue p) {
            char[] other = p.v_index.replaceAll("[0-9]", "").toUpperCase().toCharArray();
            char[] self = this.v_index.replaceAll("[0-9]", "").toUpperCase().toCharArray();
            int self_num = 0;
            int other_num = 0;
            for (int i = 0; i < self.length; i++) {
                self_num = self_num * 26 + (self[i] - 64);
            }
            for (int i = 0; i < other.length; i++) {
                other_num = other_num * 26 + (other[i] - 64);
            }
            return self_num - other_num;
        }
    }

    /**
     * 获取真实的数据（处理空格）
     */
    public List<String> getMyDataList() throws Exception {
        List<String> myDataList = new ArrayList<>();
        if (rowData == null || rowData.size() <= 0) {
            return myDataList;
        }

        int j = 0;
        for (; j < rowData.size() - 1; j++) {
            //获取当前值,并存储
            IndexValue current = rowData.get(j);
            myDataList.add(current.v_value);
            //预存下一个
            IndexValue next = rowData.get(j + 1);
            //获取差值
            int level = next.getLevel(current);
            if (level <= 0) {
                throw new Exception("超出处理范围");
            }
            for (int k = 0; k < level - 1; k++) {
                myDataList.add("");
            }
        }
        myDataList.add(rowData.get(j).v_value);
        if (currentRow == 0) {
            maxColNum = myDataList.size();
        }
        for (int i = myDataList.size(); i < maxColNum; i++) {
            myDataList.add("");
        }
        return myDataList;
    }

    public static void main(String[] args) throws Exception {
        File file = new File("D:\\test\\zjctest07.xlsx");
        ExcelRowReaderImpl excelRowReader = new ExcelRowReaderImpl();
        ExcelReader07EventModel test = new ExcelReader07EventModel(excelRowReader);
        test.processOneSheet(file, 0);
    }
}
```

## 4. 工具类

```
public class ExcelReaderUtil {
    public static final String EXCEL03_EXTENSION = ".xls";
    public static final String EXCEL07_EXTENSION = ".xlsx";

    // sheetnum从0开始
    public static void readExcel(ExcelRowReaderInterface reader, File file, int sheetnum) throws Exception {
        String fileName = file.getName();
        if (fileName.endsWith(EXCEL03_EXTENSION)) {
            ExcelReader03EventModel excelReader03EventModel = new ExcelReader03EventModel(reader);
            excelReader03EventModel.processOneSheet(file, sheetnum);
        } else if (fileName.endsWith(EXCEL07_EXTENSION)) {
            ExcelReader07EventModel excelReader07EventModel = new ExcelReader07EventModel(reader);
            excelReader07EventModel.processOneSheet(file, sheetnum);
        } else {
            throw new Exception("文件格式错误，fileName的扩展名只能是xls或xlsx。");
        }
    }

    public static void readExcel(ExcelRowReaderInterface reader, File file) throws Exception {
        String fileName = file.getName();
        if (fileName.endsWith(EXCEL03_EXTENSION)) {
            ExcelReader03EventModel excelReader03EventModel = new ExcelReader03EventModel(reader);
            excelReader03EventModel.processAllSheet(file);
        } else if (fileName.endsWith(EXCEL07_EXTENSION)) {
            ExcelReader07EventModel excelReader07EventModel = new ExcelReader07EventModel(reader);
            excelReader07EventModel.processAllSheet(file);
        } else {
            throw new Exception("文件格式错误，fileName的扩展名只能是xls或xlsx。");
        }
    }

    public static void main(String[] args) throws Exception {
        ExcelRowReaderInterface rowReader = new ExcelRowReaderImpl();
        ExcelReaderUtil.readExcel(rowReader, new File("D:\\test\\zjctest03.xls"), 0);
    }
}
```