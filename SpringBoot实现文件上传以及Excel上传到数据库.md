# **SpringBoot实现文件上传以及Excel上传到数据库**

## 一，文件上传

1，依赖jar包在spring-boot-start-web下

```
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

2，開啟文件上傳，配置上傳大小

```
 spring:
 	servlet:
        multipart:
          enabled: true
          max-file-size: 100MB #最大支持文件大小
          max-request-size: 100MB #最大支持请求大小
```

3，页面编写

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="upload" method="post" enctype="multipart/form-data">
    文件:<input type="file" name="file"><br>
    <input type="submit" value="上传">
</form>
</body>
</html>
```

4，controller类编写

```
@RestController
public class UploadController {

    @RequestMapping("/upload")
    public String handleFileUpload(@RequestParam("file") MultipartFile file) {
        if (file.isEmpty()) {
            return "文件为空";
        }
        // 获取文件名
        String fileName = file.getOriginalFilename();
        System.out.println("上传的文件名为：" + fileName);
        // 获取文件的后缀名
        String suffixName = fileName.substring(fileName.lastIndexOf("."));
        System.out.println("上传的后缀名为：" + suffixName);
        // 文件上传后的路径
        String filePath = "D://";
        File dest = new File(filePath + fileName);
        // 检测是否存在目录
        if (!dest.getParentFile().exists()) {
            dest.getParentFile().mkdirs();
        }
        try {
            file.transferTo(dest);
            return "上传成功";
        } catch (IllegalStateException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return "上传失败";
    }
}

```

## 二，将文件上传并存入数据库

1,2步同文件上传一致

3，html页面

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="importUser" method="post" enctype="multipart/form-data">
    文件:<input type="file" name="UploadFile"><br>
    <input type="submit" value="上传">
</form>
</body>
</html>
```

4，编写Excel数据处理工具类

```
package com.wist.utils;

import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.DateUtil;

/**
 * 处理Excel数据类型
 */
public class ExcelUtil {

    public static Object getValue(Cell cell){
        Object value=null;
        switch (cell.getCellType()){
            case STRING://处理string类型
                value=cell.getStringCellValue();
                break;
            case BOOLEAN://处理boolean类型
                value=cell.getBooleanCellValue();
                break;
            case NUMERIC://处理数值类型以及日期类型
                if (DateUtil.isCellDateFormatted(cell)){
                    value=cell.getDateCellValue();
                }else{
                    value=cell.getNumericCellValue();
                }
                break;
            case FORMULA:
                value=cell.getCellFormula();
                break;
            default:
                break;
        }
        return  value;
    }
}


```

5，Controller代码

```
package com.wist.controller;

import com.wist.pojo.User;
import com.wist.service.UserService;
import com.wist.utils.ExcelUtil;
import com.wist.utils.JsonResult;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.servlet.ModelAndView;

import java.util.ArrayList;
import java.util.List;

@RestController
public class UserController {
    @Autowired
    private UserService userService;
    @RequestMapping("/uploadA.htm")
    public ModelAndView upLoad(){
        ModelAndView modelAndView=new ModelAndView();
        modelAndView.setViewName("upload/upload");
        return modelAndView;
    }
    
    @RequestMapping("/importUser")
    public JsonResult importUser(@RequestParam("UploadFile") MultipartFile UploadFile){
        System.out.println("========="+UploadFile.getOriginalFilename());
        JsonResult jsonResult= new  JsonResult();
        try{
            //获取文件的输入流
            Workbook workbook=new XSSFWorkbook(UploadFile.getInputStream());
            Sheet sheet = workbook.getSheetAt(0);
            List<User> users=new ArrayList<>();
            int lastRowNum=sheet.getLastRowNum();//获取最后一行的索引
            for (int i=1;i<=lastRowNum;i++){
                Row row = sheet.getRow(i);
                short lastCellNum = row.getLastCellNum();// 获取最后一列的索引
                Object[] objs=new Object[lastCellNum];
                for (int j=0;j<lastCellNum;j++){
                    Cell cell = row.getCell(j);
                    Object value = ExcelUtil.getValue(cell);//通过数据处理类将获取的Excel表格数据处理
                    objs[j]=value;
                }
                User user=new User();
                user.setAge(((Double)objs[1]).intValue());//Excel取出的数值类型默认为Double,要将数据处理为与数据库一致的类型
                user.setName(objs[0].toString());
                users.add(user);
            }
            this.userService.importUser(users);
            jsonResult.setCode(200);
        }catch (Exception e){
            e.printStackTrace();
            jsonResult.setCode(500);
        }
        return jsonResult;
    }

}
```

## 三，将数据库查询的数据生成Excel并下载

1，页面

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<table border="0" style="margin-top:4px; margin-left: 18px">
<tr>
    <td><a href="#" class="easyui-linkbutton" onclick="downloadfile();">数据导出</a></td>
</tr>
</table>
</body>
<script>
    function downloadfile(){
        window.location.href="UserExcelDownloads";
    }
</script>
</html>
```

2，Controller类

```
 @RequestMapping(value = "UserExcelDownloads", method = RequestMethod.GET)
    public void downloadAllClassmate(HttpServletResponse response) throws IOException {
        XSSFWorkbook workbook = new XSSFWorkbook();
        XSSFSheet sheet = workbook.createSheet("信息表");
        List<User> classmateList = userService.findAllUser();
        String fileName = "userinf"  + ".xlsx";//设置要导出的文件的名字
        //新增数据行，并且设置单元格数据
        int rowNum = 1;

        String[] headers = {  "姓名", "年龄"};
        //headers表示excel表中第一行的表头

        XSSFRow row = sheet.createRow(0);
        //在excel表中添加表头

        for(int i=0;i<headers.length;i++){
            XSSFCell cell = row.createCell(i);
            XSSFRichTextString text = new XSSFRichTextString(headers[i]);
            cell.setCellValue(text);
        }

        //在表中存放查询到的数据放入对应的列
        for (User user : classmateList) {
            XSSFRow row1 = sheet.createRow(rowNum);
            row1.createCell(0).setCellValue(user.getName());
            row1.createCell(1).setCellValue(user.getAge());
            rowNum++;
        }

        response.setContentType("application/octet-stream");
        response.setHeader("Content-disposition", "attachment;filename=" + fileName);
        response.flushBuffer();
        workbook.write(response.getOutputStream());
    }

```

## 四，Hutool工具类实现Excel的导入和导出

导入Hutool的jar包

```
  <!--  Excel与数据库交互  -->
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>4.0.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml-schemas</artifactId>
            <version>4.0.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>4.0.1</version>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.0.3</version>
        </dependency>
```

### 导出Excel

```

​public void getComExcelService(HttpServletResponse response) {
​
    List<ZbComVo> mqSendTaskDTO = zbComMapper.selectZbProExcel();
        System.err.println(mqSendTaskDTO.toString());
        for(ZbComVo zcv:mqSendTaskDTO){
            if(zcv != null){
                ExcelWriter writer = cn.hutool.poi.excel.ExcelUtil.getWriter(true);
 
           
                writer.addHeaderAlias("comName", "企业名称");
                writer.addHeaderAlias("comQybm", "企业编码");
                writer.addHeaderAlias("comQyxz", "企业性质");
                writer.addHeaderAlias("comFrdb", "法人代表");
                writer.addHeaderAlias("comStszsf", "所在省份");
                writer.addHeaderAlias("comZz", "资质");
                writer.addHeaderAlias("comWz", "违章");
                writer.addHeaderAlias("comLxdw", "联系单位");
                writer.addHeaderAlias("comZxdz", "注册地址");
               
                writer.write(mqSendTaskDTO, true);
 
                writer.autoSizeColumnAll();
 
                response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet;charset=utf-8");
                response.setHeader("Content-Disposition","attachment;filename="+ UUID.randomUUID().toString().replace("-","")+ ".xlsx");
 
                ServletOutputStream out= null;
                try {
                    out = response.getOutputStream();
                } catch (IOException e) {
                    e.printStackTrace();
                }
 
                writer.flush(out, true);
                // 关闭writer，释放内存
                writer.close();
                //此处记得关闭输出Servlet流
                IoUtil.close(out);
            }
        }
 
    }
```

### Excel的导入

```
@PostMapping("/readExcel")
@ResponseBody
public Base  readExcel(@RequestParam("file") MultipartFile file){
  List<ZbComVo> zbComs=null;
  try{
  		InputStream inputStream=file.getInputStream();
  		zbComs=zbComService.getZbComExcelervice(inputStream);
  }catch(IOExceptioj e){
  		e.printStackTrace();
  }
}
```

```

public List<ZbComVo> getZbComExcelService(InputStream inputStream) {
        ExcelReader writer = cn.hutool.poi.excel.ExcelUtil.getReader(inputStream);
        writer.addHeaderAlias("企业名称", "comName");
        writer.addHeaderAlias("企业编码","comQybm");
        writer.addHeaderAlias("企业性质","comQyxz");
        writer.addHeaderAlias("法人代表","comFrdb");
        writer.addHeaderAlias("所在省份","comStszsf");
        writer.addHeaderAlias("资质","comZz");
        writer.addHeaderAlias("违章","comWz");
        writer.addHeaderAlias("联系单位","comLxdw");
        writer.addHeaderAlias("注册地址","comZxdz");
       
        List<ZbComVo> zbComs = writer.readAll(ZbComVo.class);
        System.out.println(zbComs.toString());
       return zbComs;
}

```

