---
layout: post
title:  "Java实现将查询结果导出成excel文件"
imges: 
date:   2020-11-16 10:02:00 +0800
description: 
tags: java SpringBoot
---

# 将java查询的list集合生成excel文件，前端下载

**本文介绍笔者在测试中将java查询得到的list集合通过工具类生成excel文件，返回给前端下载**

## 前端查询

###  1. 前端的请求
<br>

![](https://gitee.com/zengyimingming/picrepo/raw/master/images/20201116115420.png)

上面是前端的请求地址http://localhost:10498/boss/v1/customer/exportCustomerList 使用POST的请求方式，然后请求参数现在是为空的

### 2. 请求后返回的结果
<br>

然后是查询出的结果，这里笔者先给出请求后的结果，参考者看了结果后有助于理解下面笔者做出的具体的实现方式

<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20201116115809.png" style="zoom:50%;" />

这个请求后，下载得到的excel表

打开查看如下图，返回的结果笔者打了马赛克(数据有必要马赛克)

![](https://gitee.com/zengyimingming/picrepo/raw/master/images/20201116120126.png)

对于前端使用什么技术没关系，笔者前段使用Vue

## 后端实现流程

使用和导出excel有关的工具包

### 1.导入相关依赖

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>3.17</version>
</dependency>
```

### 2. 导入生成Excel的工具类ExcelUtil.java
<br>

```java
package com.chaoqi.payplatform.boss.util;

import org.apache.poi.hssf.usermodel.*;
import org.apache.poi.hssf.util.HSSFColor;
import org.apache.poi.ss.usermodel.BorderStyle;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.CellStyle;
import org.apache.poi.ss.util.CellRangeAddress;

import javax.servlet.http.HttpServletResponse;
import javax.swing.border.Border;
import java.io.IOException;
import java.util.List;

/**
 * Excel工具类
 */
public class ExcelUtil {

    /**
     * Excel表格导出
     *
     * @param response    HttpServletResponse对象
     * @param excelData   Excel表格的数据，封装为List<List<String>>
     * @param sheetName   sheet的名字
     * @param fileName    导出Excel的文件名
     * @param columnWidth Excel表格的宽度，建议为15
     * @throws IOException 抛IO异常
     */
    public static void exportExcel(HttpServletResponse response,
                                   List<List<String>> excelData,
                                   String sheetName,
                                   String fileName,
                                   int columnWidth) throws IOException {
        //声明一个工作簿
        HSSFWorkbook workbook = new HSSFWorkbook();
        //生成一个表格，设置表格名称
        HSSFSheet sheet = workbook.createSheet(sheetName);
        //------------------设置样式测试----------------------------
        HSSFFont font1 = workbook.createFont();
        font1.setFontName("微软雅黑");
        font1.setBold(true);
        font1.setFontHeightInPoints((short) 12);
        HSSFCellStyle style1 = workbook.createCellStyle();
        style1.setFont(font1);
        //------------------end of设置样式测试----------------------
        //设置表格列宽度
        sheet.setDefaultColumnWidth(columnWidth);
		//设置第一列的宽度
        sheet.setColumnWidth(0, (int)((10 + 0.72) * 256));

        //写入List<List<String>>中的数据
        int rowIndex = 0;

        for (List<String> data : excelData) {
            //创建一个row行，然后自增1
            HSSFRow row = sheet.createRow(rowIndex++);
            //遍历添加本行数据
            for (int i = 0; i < data.size(); i++) {
                //创建一个单元格
                HSSFCell cell = row.createCell(i);
                //创建一个内容对象
                HSSFRichTextString text = new HSSFRichTextString(data.get(i));
                //将内容对象的文字内容写入到单元格中
                cell.setCellValue(text);
            }
            if (1 == rowIndex) {
                //row.setHeight((short) 500); //设置行高
                //这里设置首行字体为自定义的style
                for (int j = 0; j < data.size(); j++) {
                    style1.setBorderBottom(BorderStyle.THIN);
                    style1.setBorderLeft(BorderStyle.THIN);
                    style1.setBorderRight(BorderStyle.THIN);
                    row.getCell(j).setCellStyle(style1);
                }
            }
        }
        //准备将Excel的输出流通过response输出到页面下载
        //八进制输出流
        response.setContentType("application/octet-stream");

        //设置导出Excel的名称
        response.setHeader("Content-disposition", "attachment;filename=" + fileName);

        //刷新缓冲
        response.flushBuffer();

        //workbook将Excel写入到response的输出流中，供页面下载该Excel文件
        workbook.write(response.getOutputStream());

        //关闭workbook
        workbook.close();
    }

}

```

### 3.控制层

为了方便显示，笔者直接将业务代码放在了控制层，对于service层和dao层的实现其实就是层层调用，直到调用mapper执行数据库的查询操作,post请求查询的参数都封装在了名为CustomerDO的实体类中了

```java
//导出Customer excel的接口
    @RequestMapping(value = "/exportCustomerList",method = RequestMethod.POST)
    public void excelDownload(HttpServletResponse response, @RequestBody CustomerDO customerDO) throws IOException, ParseException {
        List<List<String>> excelData = new ArrayList<>();
        List<String> head = new ArrayList();
        head.add("会员ID");
        head.add("会员名称");
        head.add("推荐人");
        head.add("电话");
        head.add("会员状态");
        head.add("身份证");
        head.add("开户时间");
        head.add("统一信用社代码");
        head.add("法定代表人");
        head.add("公司地址");

        excelData.add(head);

        String sheetName = "会员信息表数据";
        String fileName = "CustomerListExport.xls";
        //不带分页条件的条件查询
        List<CustomerDO> selectCustomerList = customerService.selectwithoutpage(customerDO);
        for (CustomerDO list : selectCustomerList) {
            List<String> data = new ArrayList();
            data.add(list.getId().toString());
            data.add(list.getName());
            data.add(list.getRefereer());
            data.add(list.getTel());
            if(0==list.getStatus()){
                data.add("未激活");
            }else{
                data.add("已激活");
            }
            data.add(list.getIdCard());
            //格式化时间
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
            data.add(sdf.format(list.getCreateTime()));
            data.add(list.getUsci());
            data.add(list.getLegalPerson());
            data.add(list.getCompanyAddress());
            excelData.add(data);
        }
        ExcelUtil.exportExcel(response, excelData, sheetName, fileName, 20);
    }
```

### 4. 建议

笔者在上面没有给出service层和dao层的代码实现，其实笔者在项目里面的service层和dao层都没有什么业务代码滴，你只需要实现如下语句的具体实现就行啦

```java
List<CustomerDO> selectCustomerList = customerService.selectwithoutpage(customerDO);
```

就是一个简单的对mysql的条件查询，最后返回的一个对应实体类的List集合就行啦，对于想要实现自定义的excel格式，你需要在上面的ExcelUtils.java文件添加代码，具体的修改还是要找度娘。












