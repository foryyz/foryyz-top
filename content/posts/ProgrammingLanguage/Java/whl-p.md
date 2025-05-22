---
title: '期末JavaWeb作业模板 - 2024大三上'
date: 2024-12-05T15:34:19+08:00
draft: false
---

## 0 文件结构

Data - 数据对象类

DataDao - 数据对象处理方法类

- `int getRowCount();`	// 获取数据总条数
  - int getRowCount(String s_data_name, String s_data_tag); // 获取特定数据的总条数
- `int insertPicture(Data data);` // 添加数据
- `void putVarTool(Data data, ResultSet rs);` // 数据库读取单条数据
- `List<Data> findAllData(int pageSize, int pageNow);` // 返回所有数据
  - `List<Data> findAllData(String s_data_name, String s_data_tag, int pageSize, int pageNow);` // 按照name和tag返回数据
- `Data getDataById(int id);` // 根据id查询特定数据
- `int updateDataById(int id, Data data);` // 按id修改特定数据



## 1 查、排序、分页、搜索

### 1.0 初始化数据

```java
<%
  DataDao dataDao = new DataDao();
  List<Data> dataList;
  // 获取该页数据
  String sort = request.getParameter("sort");
  String order = request.getParameter("order");
  String s_name = request.getParameter("s_name");
  String s_tag = request.getParameter("s_tag");
  int rowCount = dataDao.getRowCount(s_name,s_tag); // 图片总数
  int pageSize = 3; // 每页显示条数
  int pageNow = 1; // 当前页码
  int pageCount = 0; // 总页数
  // 计算一共多少页
  if (rowCount % pageSize == 0){
    pageCount = rowCount/pageSize;
  }else {
    pageCount = rowCount/pageSize +1;
  }
  // 接收更新的页码
  String inputPageNow = request.getParameter("page_now");
  if (inputPageNow != null){
    pageNow = Integer.parseInt(inputPageNow);
  }

  if (sort != null && order!=null && !sort.equals("null") && !order.equals("null")){
    dataList = dataDao.findAllData(sort, order, s_name, s_tag, pageSize, pageNow);
  }else {
    dataList = dataDao.findAllData(s_name, s_tag, pageSize, pageNow);
  }
%>
```

### 1.1 条目显示相关

条目, 搜索并找到合适的 \<thead\>，将代码替换.

```html
      <tr class="text-c">
        <th width="80">ID</th>
        <th width="100">昵称</th>
        <th width="40">类型</th>
        <th width="140"><a href="?s_tag=<%=request.getParameter("s_tag")%>&s_name=<%=request.getParameter("s_name")%>&sort=picture_time&order=<%= "asc".equals(request.getParameter("order")) ? "desc" : "asc" %>">时间↕️</a></th>
        <th width="70">状态</th>
        <th width="100">操作</th>
      </tr>
```

条目, 搜索并找到合适的 \<tbody\>，将代码替换.

```html
      <%for (int i=0; i<dataList.size(); ++i){
        Data data = dataList.get(i);
      %>
    <tr class="text-c">
      <td><%=data.getId()%></td>
      <td><%=data.getName()%></td>
      <td><%=data.getTag()%></td>
      <td><%=data.getTime()%></td> <input type="hidden" id="timeSortOrder" value="<%= request.getParameter("order") != null ? request.getParameter("order") : "asc" %>">
      <%if(data.getStatus() == 1) {%><td class="td-status"><span class="label label-success radius">已启用</span></td><%}%>
      <%if(data.getStatus() == 0) {%><td class="td-status"><span class="label label-default radius">未启用</span></td><%}%>
      <td class="td-manage">
        <a title="查看" href="./<%=Data.base_show_jsp%>?id=<%=data.getId()%>" class="ml-5" style="text-decoration:none"><i class="Hui-iconfont">&#xe665;</i></a>
        <a title="编辑" href="./<%=Data.base_list_jsp%>?id=<%=data.getId()%>" class="ml-5" style="text-decoration:none"><i class="Hui-iconfont">&#xe6df;</i></a>
        <a title="删除" href="javascript:;" onclick="data_del(this,<%=data.getId()%>)" class="ml-5" style="text-decoration:none"><i class="Hui-iconfont">&#xe6e2;</i></a>
      </td>
    </tr>
      <%}%>
```

### 1.2 分页相关：

在确保读取到初始化数据的位置后填入

```html
    <span>
            |         |
            >>>
            <%if (pageNow != 1){%><a href="<%=Data.base_list_jsp%>?page_now=1&s_name=<%=s_name%>&s_tag=<%=s_tag%>&sort=<%=sort%>&order=<%=order%>">首页</a><%}%>
            <%if (pageNow > 1){%><a href="<%=Data.base_list_jsp%>?page_now=<%=pageNow-1%>&s_name=<%=request.getParameter("s_name")%>&s_tag=<%=request.getParameter("s_tag")%>&sort=<%=request.getParameter("sort")%>&order=<%=request.getParameter("order")%>">上一页</a><%}%>
            第 <%=pageNow%>/<%=pageCount%> 页
            <%if (pageNow < pageCount){%><a href="<%=Data.base_list_jsp%>?page_now=<%=pageNow+1%>&s_name=<%=request.getParameter("s_name")%>&s_tag=<%=request.getParameter("s_tag")%>&sort=<%=request.getParameter("sort")%>&order=<%=request.getParameter("order")%>">下一页</a><%}%>
            <%if (pageNow != pageCount){%><a href="<%=Data.base_list_jsp%>?page_now=<%=pageCount%>&s_name=<%=s_name%>&s_tag=<%=s_tag%>&sort=<%=sort%>&order=<%=order%>">尾页</a><%}%>
            <<<
            |         |
            <input type=text name=pageGo value="<%=pageNow%>" class="SubmitStyle">
            <a href="javascript:pageGo();">转到</a>
        </span>
```



跳转相关逻辑，在<%=Data.base_list_jsp%>最下方添加

```html
<script>
  function getParameterByName(name, url) {
    if (!url) url = window.location.href;
    name = name.replace(/[$$]/g, '\\$&');
    var regex = new RegExp('[?&]' + name + '(=([^&#]*)|&|#|$)'),
            results = regex.exec(url);
    if (!results) return null;
    if (!results[2]) return '';
    return decodeURIComponent(results[2].replace(/\+/g, ' '));
  }

  function pageGo(){
    var pagoGo = document.getElementsByName("pageGo")[0].value;
    // console.log(pagoGo);
    if (isNaN(pagoGo) || pagoGo<1 || pagoGo><%=pageCount%>){
      alert("请输入正确的页码！")
      return;
    }else {
      // 构造新的URL
      var pageNow = pagoGo; // 默认值为1
      var s_name = getParameterByName('s_name') || 'null'; // 默认值为空字符串
      var s_tag = getParameterByName('s_tag') || 'null'; // 默认值为空字符串
      var sort = getParameterByName('sort') || 'null'; // 默认值为空字符串
      var order = getParameterByName('order') || 'null'; // 默认值为空字符串

      // 构造新的URL
      var newUrl = "<%=Data.base_list_jsp%>?page_now=" + encodeURIComponent(pageNow) +
              "&s_name=" + encodeURIComponent(s_name) +
              "&s_tag=" + encodeURIComponent(s_tag) +
              "&sort=" + encodeURIComponent(sort) +
              "&order=" + encodeURIComponent(order);

      // 执行重定向
      window.location.href = newUrl;
    }
  }
</script>
```

### 1.3 搜索相关：

```html
  <div class="text-c"> 搜索条件：
    <input type="text" name="search_tag" id="search_tag"
      <%if (request.getParameter("s_tag") != null && !request.getParameter("s_tag").equals("null")){%>
           value="<%=request.getParameter("s_tag")%>"<%}%>
           placeholder=" 标签" style="width:250px" class="input-text">
    <input type="text" name="search_name" id="search_name" placeholder=" 名称" style="width:250px" class="input-text">
    <a href="#" onclick="return dataSearch();"> <button name="" id="" class="btn btn-success" type="submit"><i class="Hui-iconfont">&#xe665;</i> 搜索</button> </a>
  </div>
```

### 1.4 其他js

在head标签中写入js代码

```html
  <script>
    function sortByTime(){
      // 默认排序规则，可以通过点击按钮切换
      let currentOrder = document.getElementById("timeSortOrder").value;
      let newOrder;
      if (currentOrder == "asc"){
        newOrder = "desc";
      }else{
        newOrder = "asc";
      }
      document.getElementById("timeSortOrder").value = newOrder;
      // 发起请求（使用 AJAX 或直接重定向）
      window.location.href = `<%=Data.base_list_jsp%>?sort=time&order=` + newOrder;
    }

    function dataSearch(){
      var tag = document.getElementById('search_tag').value;
      var name = document.getElementById('search_name').value;
      var url = '<%=Data.base_list_jsp%>';
      if (tag !== "" && name !== ""){
        url = '<%=Data.base_list_jsp%>?s_name=' + encodeURIComponent(name) + '&s_tag=' + encodeURIComponent(tag);
      }else if(tag !== "" && name === ""){
        url = '<%=Data.base_list_jsp%>?s_tag=' + encodeURIComponent(tag);
      }else if(name !== "" && tag === ""){
        url = '<%=Data.base_list_jsp%>?s_name=' + encodeURIComponent(name);
      }
      window.location.href = url;
      return false; // Prevent the default action of the link
    }
  </script>
```

## 2 增 删 改

### 2.1 增

#### list.jsp

添加js代码

```js
  function data_add(title,url){
    var index = layer.open({
      type: 2,
      title: title,
      content: url
    });
    layer.full(index);
  }
```

在合适位置添加按钮

```html
<a href="javascript:;" onclick="data_add('添加','<%=Data.base_add_jsp%>')" class="btn btn-primary radius"><i class="Hui-iconfont">&#xe600;</i> 添加</a>
```



#### add.jsp

Head 标签 js

```html
  <style>
    @keyframes shake {
      0% { transform: translate(0, 0); }
      25% { transform: translate(-5px, 0); }
      50% { transform: translate(5px, 0); }
      75% { transform: translate(-5px, 0); }
      100% { transform: translate(0, 0); }
    }
    /* 默认静止状态 */
    body {
      transition: transform 0.2s;
    }

    /* 添加震动效果的类 */
    .shake {
      animation: shake 0.5s ease-in-out;
    }
  </style>
  <script>
    let nameFlag = false;
    let is_null = false;
    let timeFlag = false; // 判断图片时间是否添加

    function shakePage() {
      // 给 body 添加震动效果的类
      document.body.classList.add('shake');
      // 在动画结束后移除类
      setTimeout(() => {
        document.body.classList.remove('shake');
      }, 500); // 动画时长需要与 CSS 中定义的时间一致
    }
    function checkName(){
      nameFlag = false;
      var fmName = fm.name.value;
      var name_inputElement = document.getElementById("name");

      if (fmName === ""){
        name_inputElement.placeholder = "请输入昵称！";
        name_inputElement.style.border = "1px solid brown"
      } else{
        // 异步校验
        var xmlHttp = new XMLHttpRequest();
        xmlHttp.open("post", "DataNameCheckServlet?name=" + fmName, true);
        xmlHttp.onreadystatechange = function (){
          var rs = xmlHttp.responseText;
          if (xmlHttp.readyState === 4 && xmlHttp.status === 200){
            if (rs == "fail"){
              name_inputElement.style.border = "1px solid brown"
              document.getElementById("checkNameSpan").style.color = "brown";
              document.getElementById("checkNameSpan").innerHTML = "该昵称已存在!";
              shakePage();
            }else{
              name_inputElement.placeholder = ""; // 恢复默认 placeholder
              name_inputElement.style.border = ""; // 移除边框样式
              document.getElementById("checkNameSpan").innerHTML = "";
              nameFlag = true;
            }
          }
        }
        xmlHttp.send(null);
      }
    }
    function isNull(){
      is_null = true;
      var tag_inputElement = document.getElementById("tag");
      var fmTag = fm.tag.value;
      if (fmTag === ""){
        tag_inputElement.placeholder = "请输入类型！";
        tag_inputElement.style.border = "1px solid brown"
        is_null = false;
      }else{
        tag_inputElement.placeholder = ""; // 恢复默认 placeholder
        tag_inputElement.style.border = ""; // 移除边框样式
      }

      var status_inputElement = document.getElementById("status");
      var fmStatus = fm.status.value;
      if (fmStatus === ""){
        document.getElementById("checkStatusSpan").style.color = "brown";
        document.getElementById("checkStatusSpan").innerHTML = "请选择状态!";
        is_null = false;
      }else{
        document.getElementById("checkStatusSpan").innerHTML = "";
      }

      var fmName = fm.name.value;
      var name_inputElement = document.getElementById("name");

      if (fmName === ""){
        name_inputElement.placeholder = "请输入昵称！";
        name_inputElement.style.border = "1px solid brown"
        is_null = false;
      }
    }

    // 提交表单
    function fm_submit() {
      isNull();
      if (nameFlag && is_null){
        timeFlag = false; // 开始判断时间是否添加
        var fmTime = fm.time.value;
        if (fmTime === ""){
          fm.time.value = "-1"; // 标记为未输入Time并传给后端
        }else{
          timeFlag = true;
        }
        fm.submit();
        if (timeFlag){
          alert("添加成功！");
        }else{
          alert("添加成功！自动设置添加时间为当前时间");
        }

      }
      else{
        shakePage();
        return false;
      }
    }
  </script>
```

from表单 一般在\<article\>标签内

```html
<form name="fm" action="./DataAddServlet" method="post" class="form form-horizontal" id="form-data-add">
    <div class="row cl">
      <label class="form-label col-xs-4 col-sm-3"><span class="c-red">*</span>昵称：</label>
      <div class="formControls col-xs-8 col-sm-9">
        <input type="text" class="input-text" value="" placeholder="" id="name" name="name" onblur="checkName();">
      </div>
    </div>
    <div class="row cl">
      <label class="form-label col-xs-4 col-sm-3"><span class="c-red">*</span>类型：</label>
      <div class="formControls col-xs-8 col-sm-9">
        <input type="text" class="input-text" value="" placeholder="" id="tag" name="tag" >
      </div>
    </div>
    <div class="row cl">
      <label class="form-label col-xs-4 col-sm-3">时间：</label>
      <div class="formControls col-xs-8 col-sm-9">
        <input type="text" class="input-text" placeholder="year-month-day" name="time" id="time">
      </div>
    </div>
    <div class="row cl">
      <label class="form-label col-xs-4 col-sm-3">状态：</label>
      <div class="formControls col-xs-8 col-sm-9"> <span class="select-box">
				<select class="select" size="1" name="status">
					<option value="" selected>请选择状态</option>
					<option value="0">已通过</option>
					<option value="1">未批准</option>
				</select>
				</span> </div>
      <span id="checkStatusSpan"></span></br>
    </div>
    <div class="row cl">
      <div class="col-xs-8 col-sm-9 col-xs-offset-4 col-sm-offset-3">
        <span id="checkNameSpan"></span></br>
        <button onClick="fm_submit();" class="btn btn-primary radius" type="button"><i class="Hui-iconfont">&#xe632;</i> 提交</button>
      </div>
    </div>
  </form>
```

### 2.2 删除

list.jsp 末尾 添加js代码

```js
/*删除*/
    function data_del(obj, id){
    layer.confirm('确认要删除吗？', function (index) {
      $.ajax({
        type: 'POST',
        url: './DataDelServlet?id=' + id,
        dataType: 'text',
        success: function () {
          $(obj).parents("tr").remove();
          layer.msg('已删除!', {icon: 1, time: 1000});
        },
        error: function () {
          alert("删除失败！")
        },
      });
    });
  }
```

### 2.3 改

复制add.jsp 改名为 edit.jsp，将Head标签下的JS语法全部删除。并：

\<body\> 标签下添加

```html
<%
  Data data = new Data();
  DataDao dao = new DataDao();
  data = dao.getDataById(Integer.parseInt(request.getParameter("id")));
%>
```

**以前添加的** \<style> 和 \<script\> 改为:

```html
  <style>
    @keyframes shake {
      0% { transform: translate(0, 0); }
      25% { transform: translate(-5px, 0); }
      50% { transform: translate(5px, 0); }
      75% { transform: translate(-5px, 0); }
      100% { transform: translate(0, 0); }
    }
    /* 默认静止状态 */
    body {
      transition: transform 0.2s;
    }

    /* 添加震动效果的类 */
    .shake {
      animation: shake 0.5s ease-in-out;
    }
  </style>
  <script>
    let is_null = false;
    let timeFlag = false; // 判断图片时间是否添加

    function shakePage() {
      // 给 body 添加震动效果的类
      document.body.classList.add('shake');
      // 在动画结束后移除类
      setTimeout(() => {
        document.body.classList.remove('shake');
      }, 500); // 动画时长需要与 CSS 中定义的时间一致
    }

    function isNull(){
      is_null = true;
      var tag_inputElement = document.getElementById("tag");
      var fmTag = fm.tag.value;
      if (fmTag === ""){
        tag_inputElement.placeholder = "请输入类型！";
        tag_inputElement.style.border = "1px solid brown"
        is_null = false;
      }else{
        tag_inputElement.placeholder = ""; // 恢复默认 placeholder
        tag_inputElement.style.border = ""; // 移除边框样式
      }
      
      var fmStatus = fm.status.value;
      if (fmStatus === ""){
        document.getElementById("checkStatusSpan").style.color = "brown";
        document.getElementById("checkStatusSpan").innerHTML = "请选择状态!";
        is_null = false;
      }else{
        document.getElementById("checkStatusSpan").innerHTML = "";
      }
      
    }

    // 提交表单
    function fm_submit() {
      isNull();
      if (is_null){
        timeFlag = false; // 开始判断时间是否添加
        var fmTime = fm.time.value;
        if (fmTime === ""){
          fm.time.value = "-1"; // 标记为未输入Time并传给后端
        }else{
          timeFlag = true;
        }
        fm.submit();
        if (timeFlag){
          alert("修改成功！");
        }else{
          alert("修改成功！自动设置添加时间为当前时间");
        }

      }
      else{
        shakePage();
        return false;
      }
    }
  </script>
```



\<form\>标签改为 (以前的全部删掉)

```html
  <form name="fm" action="./DataEditServlet" method="post" class="form form-horizontal" id="form-data-add">
    <input type="hidden" id="id" name="id" value="<%=data.getId()%>">
    <div class="row cl">
      <label class="form-label col-xs-4 col-sm-3">(不可改)昵称：</label>
      <div class="formControls col-xs-8 col-sm-9">
        <input type="text" class="input-text" value="<%=data.getName()%>" placeholder="" id="name" name="name"  readonly>
      </div>
    </div>
    <div class="row cl">
      <label class="form-label col-xs-4 col-sm-3">类型：</label>
      <div class="formControls col-xs-8 col-sm-9">
        <input type="text" class="input-text" value="<%=data.getTag()%>" placeholder="" id="tag" name="tag" >
      </div>
    </div>
    <div class="row cl">
      <label class="form-label col-xs-4 col-sm-3">时间：</label>
      <div class="formControls col-xs-8 col-sm-9">
        <input type="text" class="input-text" value="<%=data.getTime()%>" placeholder="year-month-day" name="time" id="time" >
      </div>
    </div>
    <div class="row cl">
      <label class="form-label col-xs-4 col-sm-3">状态：</label>
      <div class="formControls col-xs-8 col-sm-9"> <span class="select-box">
				<select class="select" size="1" name="status">
                    <%if (data.getStatus() == 1){%><option value="1">已启用</option><%}%>
                    <%if (data.getStatus() == 0){%><option value="0">未启用</option><%}%>
					<%if(data.getStatus() != 0){%><option value="0">未启用</option><%}%>
					<%if(data.getStatus() != 1){%><option value="1">已启用</option><%}%>
				</select>
				</span> </div>
      <span id="checkStatusSpan"></span></br>
    </div>
    <div class="row cl">
      <div class="col-xs-8 col-sm-9 col-xs-offset-4 col-sm-offset-3">
        <span id="checkNameSpan"></span></br>
        <button onClick="fm_submit();" class="btn btn-primary radius" type="button"><i class="Hui-iconfont">&#xe632;</i> 提交</button>
      </div>
    </div>
  </form>
```



### 2.4 查

新建show.jsp，复制自edit.jsp 

将from表单内改为 并将所有js语法删除

```html
  <form name="fm"  class="form form-horizontal" id="form-data-add">
    <div class="row cl">
      <label class="form-label col-xs-4 col-sm-3">ID：</label>
      <div class="formControls col-xs-8 col-sm-9">
        <input type="text" class="input-text" value="<%=data.getId()%>" id="id" name="id"  readonly>
      </div>
    </div>
    <div class="row cl">
      <label class="form-label col-xs-4 col-sm-3">昵称：</label>
      <div class="formControls col-xs-8 col-sm-9">
        <input type="text" class="input-text" value="<%=data.getName()%>" id="name" name="name"  readonly>
      </div>
    </div>
    <div class="row cl">
      <label class="form-label col-xs-4 col-sm-3">类型：</label>
      <div class="formControls col-xs-8 col-sm-9">
        <input type="text" class="input-text" value="<%=data.getTag()%>" id="tag" name="tag" readonly>
      </div>
    </div>
    <div class="row cl">
      <label class="form-label col-xs-4 col-sm-3">时间：</label>
      <div class="formControls col-xs-8 col-sm-9">
        <input type="text" class="input-text" value="<%=data.getTime()%>" name="time" id="time" readonly>
      </div>
    </div>
    <div class="row cl">
      <label class="form-label col-xs-4 col-sm-3">状态：</label>
      <div class="formControls col-xs-8 col-sm-9">
        <%if (data.getStatus() == 1){%><input type="text" class="input-text" value="已启用" placeholder="year-month-day" name="status" id="status" readonly><%}%>
        <%if (data.getStatus() == 0){%><input type="text" class="input-text" value="未启用" placeholder="year-month-day" name="status" id="status" readonly><%}%>
      </div>
    </div>
    <div class="row cl">
      <div class="col-xs-8 col-sm-9 col-xs-offset-4 col-sm-offset-3">
        <span id="checkNameSpan"></span></br>
        <button onClick="window.location.href=document.referrer" class="btn btn-primary radius" type="button"><i class="Hui-iconfont">&#xe632;</i> 返回</button>
      </div>
    </div>
  </form>
```

