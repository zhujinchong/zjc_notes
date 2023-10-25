1、引入依赖

```
<!--pagehelper-->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.10</version>
</dependency>
```



2、PageHelper插件会接受参数，并将参数自动加入到下一个查询数据库操作。

如果前端用的是DataTables插件，还需要扩展PageInfo返回数据类

```
import com.github.pagehelper.PageInfo;

// 供前端DataTables用
public class DataGrid<T> extends PageInfo<T> {

    public DataGrid(List<T> list) {
        super(list);
        this.recordsTotal = getTotal();
        this.recordsFiltered = getTotal();
        this.data = getList();
    }

    private long recordsTotal;

    private long recordsFiltered;

    private List<T> data;

    public long getRecordsTotal() {
        return recordsTotal;
    }

    public long getRecordsFiltered() {
        return recordsFiltered;
    }

    public List<T> getData() {
        return data;
    }


    public void setRecordsTotal(long recordsTotal) {
        this.recordsTotal = recordsTotal;
    }

    public void setRecordsFiltered(long recordsFiltered) {
        this.recordsFiltered = recordsFiltered;
    }

    public void setData(List<T> data) {
        this.data = data;
    }
}
```

3、后台使用

```
@RequestMapping(value = "/category/list", method = RequestMethod.POST)
@ResponseBody
public DataGrid<Category> getCategoryListByUser(HttpServletRequest request, HttpSession session) {
    int start = Integer.parseInt(request.getParameter("start"));
    int length = Integer.parseInt(request.getParameter("length"));
    PageHelper.startPage((start / length) + 1, length);
    User loginUser = (User) session.getAttribute("loginUser");
    if (!StringUtils.isEmpty(loginUser)) {
        List<Category> categoryList = categoryService.getCategoryByUserId(loginUser.getId());
        DataGrid<Category> categoryPageInfo = new DataGrid<>(categoryList);
        return categoryPageInfo;
    }
    return null;
}
```



4、前端使用

```
initCategoryListTable: function () {
	$('#dataListTable').dataTable({
        processing: true,
        serverSide: true,
        searching: false,//是否开始本地搜索
        autoWidth: false,
        pagingType: "full_numbers",//除首页、上一页、下一页、末页四个按钮还有页数按钮
        info: true, // 左下角显示记录数
        pageLength: 10,         // 初始长度
        lengthMenu: [10, 20, 50, 100], // 可选长度
        destroy: true, //Cannot reinitialise DataTable,解决重新加载表格内容问题
        ajax: {
            url: '${ctx}/category/list',
            method: 'post',
        },
        columns: [
            {
                data: 'id',
                title: 'No.',
                orderable: true,
                render: function (data) {
                    return data;
                }
            },
            {
                data: 'name',
                title: 'Category',
                render: function (data) {
                    return data;
                }
            },
            {
                data: 'id',
                title: 'Options',
                render: function (data, display, row) {
                    let op = '';
                    op += '<i class="bi bi-pencil-fill text-primary mr-3" title="修改" onclick="BlogManage.bindEvent.categoryUpdate(' + row.id + ',\''+row.name+'\')"></i>';
                    op += '<i class="bi bi-trash-fill text-primary" title="删除" onclick="BlogManage.bindEvent.categoryDelete(' + row.id + ')"></i>';
                    return op;
                }
            },
        ],
	});
}
```

