# 1. 前端使用js的BLOB对象分片

```
let curFile = $("#crmImportForm #importFile")[0].files[0];
let curFilename = new Date().getTime() + curFile.name;
let mergeUrl = '${ctx}/tools/file/uploader';
if (uploadFile(curFile, curFilename, mergeUrl)){
 // 成功
}


uploadFile: function (blob, filename, url) {
    let bytesPerPiesce = 1024 * 1024 * 10; // 10M
    let filesize = blob.size;
    let start = 0;
    let end;
    let index = 0;
    while (start < filesize) {
        end = start + bytesPerPiesce
        if (end > filesize) {
            end = filesize;
        }
        let chunk = blob.slice(start, end); // 分片
        let formData = new FormData();      // 往后台传递参数
        formData.append('file', chunk);
        formData.append('filename', filename);  // filename保证每次都一样，并且唯一
        formData.append('index', index * bytesPerPiesce);
        $.ajax({
            url: url,
            type: 'POST',
            cache: false,
            data: formData,
            async: false,  // 同步上传
            processData: false,
            contentType: false,
        }).fail(function () {
            return false;
        }).success(function () {
            console.log(filename + ": " + index);
        });
        start = end;
        index += 1;
    }
    return true;
},
```

# 2. 后端再合并文件

```
@PostMapping(value = "/file/uploader")
@ResponseBody
public void webUploaderImport(@RequestParam("file") MultipartFile multipartFile, String filename, Long index) {
    RandomAccessFile accessFile = null;
    BufferedInputStream inputStream = null;
    try {
     // fileStorePath文件存储路径
        File file = new File(fileStorePath, filename);
        accessFile = new RandomAccessFile(file, "rw");
        // 类似于指针定位
        accessFile.seek(index);
        inputStream = new BufferedInputStream(multipartFile.getInputStream());
        byte[] buf = new byte[1024];
        int length = 0;
        while ((length = inputStream.read(buf)) != -1) {
            accessFile.write(buf, 0, length);
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        try {
            if (inputStream != null) {
                inputStream.close();
            }
            if (accessFile != null) {
                accessFile.close();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```