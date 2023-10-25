# 1. 原理

用Java流的方式，一行一行读取。

# 2. 读CSV代码

### 1. 业务接口

```
public class CsvRowReaderImpl implements CsvRowReaderInterface{
    @Override
    public void getRows(Long curRow, List<String> rowlist) {
        System.out.println(rowlist);
    }
}
```

### 2. 实现

```
public class CsvReader {

    private CsvRowReaderInterface rowReader;

    public CsvReader(CsvRowReaderInterface rowReader) {
        this.rowReader = rowReader;
    }

    public void readCsvByBufferReader(File file) {
        try {
            BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream(file), "gbk"));
            String line = null;
            Long curRow = -1L;
            while ((line = reader.readLine()) != null) {
                curRow++;
                if (!"".equals(line.trim())) {
                 // 因为csv是","分割，同时还有这样的数据""xx,yy""，因此需要逗号分割，且不能分割双引号里面的数据
                    // 如数据：test,aaa,bb,123,"hello,world",ccc,ddd
                    String[] split = line.trim().split(",(?=([^\\\"]*\\\"[^\\\"]*\\\")*[^\\\"]*$)", -1);
                    for (int i = 0; i < split.length; i++) {
                  // 去除前后端双引号
                        split[i] = split[i].replaceAll("^\"|\"$", "");
                    }
                    List<String> strings = Arrays.asList(split);
                    // 调用接口
                    rowReader.getRows(curRow, strings);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        CsvRowReaderImpl csvRowReader = new CsvRowReaderImpl();
        CsvReader csvReader = new CsvReader(csvRowReader);
        csvReader.readCsvByBufferReader(new File("D:\\test\\BTS Information.csv"));
    }
}
```

### 3. 工具类

```
public class CsvReaderUtil {
    public static void readCsv(CsvRowReaderInterface reader, File file) throws Exception {
        String fileName = file.getName();
        if (fileName.endsWith(".csv")) {
            CsvReader csvReader = new CsvReader(reader);
            csvReader.readCsvByBufferReader(file);
        } else {
            throw new Exception("文件格式错误，fileName的扩展名只能是csv。");
        }
    }

    public static void main(String[] args) throws Exception {
        CsvRowReaderImpl csvRowReader = new CsvRowReaderImpl();
        CsvReaderUtil.readCsv(csvRowReader, new File("D:\\test\\BTS Information.csv"));
    }
}
```



# 3. web文件下载

```
@RequestMapping(value = "/download/{filename}")
public void download(@PathVariable("filename") String filename, HttpServletRequest req, HttpServletResponse resp) {
    InputStream in = null;
    File file = null;
    // filename包含文件路径进行了编码，需要解码
    filename = new String(Base64.getDecoder().decode(filename.getBytes()));
    try {
        file = new File(filename);
        in = new FileInputStream(file);
        OutputStream out = resp.getOutputStream();
        String nameDownload = filename.substring(filename.lastIndexOf("\\") + 1);
        nameDownload = nameDownload.substring(nameDownload.lastIndexOf("/") + 1);
        resp.addHeader("Content-Disposition", "attachment;filename=" + nameDownload);
        resp.setContentType("application/octet-stream");
        IOUtils.copy(in, out);
    } catch (IOException e) {
        logger.error("", e);
    } finally {
        try {
            if (in != null) {
                in.close();
                //file.delete();// 删除文件.
            }
            // out不需要关
        } catch (IOException e) {
            logger.error("", e);
        }
    }
}
```

