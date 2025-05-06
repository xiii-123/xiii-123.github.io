---
title: Apache POI
published: 2025-04-29
description: 'Apache POI的基本使用'
image: ''
tags: [apache-poi]
category: 'java-web'
draft: false 
---


# Apache POI 在 Spring Boot 中操作 Excel 完全指南
## 引言
Apache POI 是一个功能强大的 Java 库，用于处理 Microsoft Office 文件，包括 Excel。在 Spring Boot 应用程序中集成 Apache POI 可以实现 Excel 文件的导入、导出和操作功能。本指南将详细介绍如何在 Spring Boot 环境中使用 Apache POI 操作 Excel 文件，包括创建、读取和修改 Excel 文档。
## Apache POI 简介
Apache POI 是一个免费开源的跨平台 Java API，用于读取和写入 Microsoft Office 格式的文件。POI 代表 "Poor Obfuscation Is Okay"，最初是为处理 Microsoft Office 文件而创建的，现在已经成为处理 Office 文件事实上的标准 Java 库。
Apache POI 可以处理以下格式的 Excel 文件：
- .xls (Excel 97-2003 格式，使用 HSSF 类)
- .xlsx (Excel 2007 及以上格式，使用 XSSF 类)
- .ods (OpenOffice 电子表格格式，使用 ODS 类)
## 环境要求
在开始使用 Apache POI 之前，请确保您的开发环境满足以下要求：
- Java 8 或更高版本 (JRE 1.8+)
- Apache POI 5.x 或更高版本 (对于 4.x 或 5.x 版本，JDK 编译必须 1.8 及以上)[[18](https://blog.csdn.net/xiaofeng_yang/article/details/138564912)]
## Apache POI 在 Spring Boot 中的依赖
要在 Spring Boot 项目中使用 Apache POI，需要在 pom.xml 文件中添加必要的依赖项。以下是最新版本的依赖配置：
```xml
<dependencies>
    <!-- Apache POI 核心依赖 -->
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi</artifactId>
        <version>5.4.1</version>
    </dependency>
    
    <!-- Apache POI OXML 支持，用于处理 .xlsx 文件 -->
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi-ooxml</artifactId>
        <version>5.4.1</version>
    </dependency>
    
    <!-- Apache XMLBeans，POI 依赖的 XML 处理库 -->
    <dependency>
        <groupId>org.apache.xmlbeans</groupId>
        <artifactId>xmlbeans</artifactId>
        <version>5.1.1</version>
    </dependency>
    
    <!-- Commons Collections，POI 依赖的集合框架 -->
    <dependency>
        <groupId>commons-collections</groupId>
        <artifactId>commons-collections</artifactId>
        <version>3.2.2</version>
    </dependency>
</dependencies>
```
这些依赖将使您能够处理 Excel 文件的各种操作。
## Spring Boot 中的文件上传配置
在处理 Excel 文件之前，需要配置 Spring Boot 应用程序以处理文件上传。在 application.properties 文件中添加以下配置：
```properties
# 配置上传文件的最大大小
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB
# 配置临时文件目录
spring.servlet.multipart.location=/tmp
```
这些配置将设置上传文件的最大大小和临时文件存储位置。
## 创建 Excel 文件
在 Spring Boot 应用程序中，可以使用 Apache POI 创建新的 Excel 文件。以下是创建 .xlsx 文件的基本步骤：
```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import java.io.FileOutputStream;
import java.io.IOException;
public class ExcelCreator {
    
    public void createNewExcelFile() {
        // 创建一个新的工作簿
        Workbook workbook = new XSSFWorkbook();
        
        // 创建一个工作表
        Sheet sheet = workbook.createSheet("Sheet1");
        
        // 创建一行
        Row row = sheet.createRow(0);
        
        // 创建单元格并添加数据
        Cell cell = row.createCell(0);
        cell.setCellValue("Hello, World!");
        
        // 将工作簿写入文件
        try (FileOutputStream fileOut = new FileOutputStream("workbook.xlsx")) {
            workbook.write(fileOut);
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 关闭工作簿
        workbook.close();
    }
}
```
上述代码创建了一个名为 "workbook.xlsx" 的新 Excel 文件，并在第一个工作表的第一行第一列添加了 "Hello, World!" 文本。
## 读取 Excel 文件
读取现有的 Excel 文件与创建新文件同样简单。以下是使用 Apache POI 读取 Excel 文件的基本步骤：
```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import java.io.FileInputStream;
import java.io.IOException;
public class ExcelReader {
    
    public void readExcelFile() {
        try (FileInputStream fis = new FileInputStream("workbook.xlsx")) {
            // 根据文件后缀自动确定使用 XSSFWorkbook 或 HSSFWorkbook
            Workbook workbook = WorkbookFactory.create(fis);
            
            // 获取第一个工作表
            Sheet sheet = workbook.getSheetAt(0);
            
            // 获取第一行
            Row row = sheet.getRow(0);
            
            // 获取第一个单元格
            Cell cell = row.getCell(0);
            
            // 输出单元格内容
            System.out.println("Cell value: " + cell.getStringCellValue());
            
            // 关闭工作簿
            workbook.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
上述代码读取名为 "workbook.xlsx" 的 Excel 文件，并输出第一个工作表第一行第一列的单元格内容。
## 在 Spring Boot 中处理文件上传
要在 Spring Boot 应用程序中处理文件上传，可以创建一个控制器来处理上传请求。以下是一个处理 Excel 文件上传的控制器示例：
```java
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.http.ResponseEntity;
import org.springframework.http.HttpStatus;
import org.springframework.beans.factory.annotation.Autowired;
import java.io.IOException;
import java.io.File;
import java.nio.file.Path;
import java.nio.file.Paths;
@RestController
@RequestMapping("/api/excel")
public class ExcelUploadController {
    
    @Autowired
    private ExcelService excelService;
    
    @PostMapping("/upload")
    public ResponseEntity<String> uploadExcelFile(@RequestParam("file") MultipartFile file) {
        try {
            // 保存上传的文件到临时目录
            String uploadDir = "upload";
            File uploadPath = new File(uploadDir);
            
            if (!uploadPath.exists()) {
                uploadPath.mkdir();
            }
            
            String fileName = file.getOriginalFilename();
            File tempFile = new File(uploadPath, fileName);
            
            // 保存文件
            file.transferTo(tempFile);
            
            // 处理 Excel 文件
            excelService.processExcelFile(tempFile);
            
            // 返回成功响应
            return new ResponseEntity<>("File processed successfully", HttpStatus.OK);
        } catch (IOException e) {
            return new ResponseEntity<>("Error processing file: " + e.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
}
```
上述代码创建了一个处理文件上传的控制器。它将上传的文件保存到 "upload" 目录，并使用 ExcelService 类来处理该文件。
## 在 Spring Boot 中处理文件下载
要在 Spring Boot 应用程序中提供 Excel 文件下载，可以创建一个控制器来生成并下载 Excel 文件。以下是一个处理 Excel 文件下载的控制器示例：
```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.http.ResponseEntity;
import org.springframework.http.MediaType;
import org.springframework.http.HttpHeaders;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
@RestController
@RequestMapping("/api/excel")
public class ExcelDownloadController {
    
    @Autowired
    private ExcelService excelService;
    
    @GetMapping("/download")
    public ResponseEntity<byte[]> downloadExcelFile() {
        try {
            // 生成 Excel 文件
            byte[] excelContent = excelService.generateExcelFile();
            
            // 设置响应头
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
            headers.setContentDispositionFormData("attachment", "report.xlsx");
            
            // 返回文件内容
            return new ResponseEntity<>(excelContent, headers, HttpStatus.OK);
        } catch (IOException e) {
            return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
}
```
上述代码创建了一个处理文件下载的控制器。它使用 ExcelService 类生成 Excel 文件内容，并将其作为附件下载。

也可以使用`HttpServletResponse`来处理文件下载。
```java
    /**导出近30天的运营数据报表
     * @param response
     **/
    public void exportBusinessData(HttpServletResponse response) {
        InputStream inputStream = this.getClass().getClassLoader().getResourceAsStream("template/test.xlsx");
        try {
            //基于提供好的模板文件创建一个新的Excel表格对象
            XSSFWorkbook excel = new XSSFWorkbook(inputStream);
            //通过输出流将文件下载到客户端浏览器中
            ServletOutputStream out = response.getOutputStream();
            excel.write(out);
            //关闭资源
            out.flush();
            out.close();
            excel.close();

        }catch (IOException e){
            e.printStackTrace();
        }
    }

```
## ExcelService 类实现
为了使代码更模块化，可以创建一个 ExcelService 类来处理与 Excel 相关的所有业务逻辑。以下是一个完整的 ExcelService 类实现：
```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.openxml4j.exceptions.InvalidFormatException;
import org.apache.poi.poifs.filesystem.POIFSFileSystem;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.ByteArrayOutputStream;
import java.util.List;
import java.util.ArrayList;
public class ExcelService {
    
    public void processExcelFile(File file) throws IOException, InvalidFormatException {
        try (InputStream fis = new FileInputStream(file)) {
            // 根据文件后缀自动确定使用 XSSFWorkbook 或 HSSFWorkbook
            Workbook workbook = WorkbookFactory.create(fis);
            
            // 获取第一个工作表
            Sheet sheet = workbook.getSheetAt(0);
            
            // 遍历所有行
            for (Row row : sheet) {
                // 遍历所有单元格
                for (Cell cell : row) {
                    switch (cell.getCellType()) {
                        case STRING:
                            System.out.print(cell.getStringCellValue() + "\t");
                            break;
                        case NUMERIC:
                            if (DateUtil.isCellDateFormatted(cell)) {
                                System.out.print(cell.getDateCellValue() + "\t");
                            } else {
                                System.out.print(cell.getNumericCellValue() + "\t");
                            }
                            break;
                        case BOOLEAN:
                            System.out.print(cell.getBooleanCellValue() + "\t");
                            break;
                        case FORMULA:
                            System.out.print(cell.getCellFormula() + "\t");
                            break;
                        default:
                            System.out.print("UNKNOWN TYPE\t");
                            break;
                    }
                }
                System.out.println();
            }
            
            // 关闭工作簿
            workbook.close();
        }
    }
    
    public byte[] generateExcelFile() throws IOException {
        // 创建一个新的工作簿
        Workbook workbook = new XSSFWorkbook();
        
        // 创建一个工作表
        Sheet sheet = workbook.createSheet("Report");
        
        // 创建表头
        Row headerRow = sheet.createRow(0);
        headerRow.createCell(0).setCellValue("ID");
        headerRow.createCell(1).setCellValue("Name");
        headerRow.createCell(2).setCellValue("Value");
        
        // 创建数据行
        for (int i = 1; i <= 5; i++) {
            Row row = sheet.createRow(i);
            row.createCell(0).setCellValue(i);
            row.createCell(1).setCellValue("Item " + i);
            row.createCell(2).setCellValue(i * 100);
        }
        
        // 将工作簿写入字节数组
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        workbook.write(bos);
        byte[] excelContent = bos.toByteArray();
        
        // 关闭工作簿
        workbook.close();
        
        return excelContent;
    }
}
```
上述代码实现了一个完整的 ExcelService 类，它包含处理和生成 Excel 文件的方法。
## 完整的 Spring Boot 应用程序
以下是一个完整的 Spring Boot 应用程序，它结合了前面介绍的所有组件：
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
@SpringBootApplication
public class ExcelApplication {
    
    @Bean
    public ExcelService excelService() {
        return new ExcelService();
    }
    
    public static void main(String[] args) {
        SpringApplication.run(ExcelApplication.class, args);
    }
}
```
```java
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.http.ResponseEntity;
import org.springframework.http.HttpStatus;
import org.springframework.beans.factory.annotation.Autowired;
import java.io.IOException;
import java.io.File;
import java.nio.file.Path;
import java.nio.file.Paths;
@RestController
@RequestMapping("/api/excel")
public class ExcelController {
    
    @Autowired
    private ExcelService excelService;
    
    @PostMapping("/upload")
    public ResponseEntity<String> uploadExcelFile(@RequestParam("file") MultipartFile file) {
        try {
            // 保存上传的文件到临时目录
            String uploadDir = "upload";
            File uploadPath = new File(uploadDir);
            
            if (!uploadPath.exists()) {
                uploadPath.mkdir();
            }
            
            String fileName = file.getOriginalFilename();
            File tempFile = new File(uploadPath, fileName);
            
            // 保存文件
            file.transferTo(tempFile);
            
            // 处理 Excel 文件
            excelService.processExcelFile(tempFile);
            
            // 返回成功响应
            return new ResponseEntity<>("File processed successfully", HttpStatus.OK);
        } catch (IOException | InvalidFormatException e) {
            return new ResponseEntity<>("Error processing file: " + e.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
    
    @GetMapping("/download")
    public ResponseEntity<byte[]> downloadExcelFile() {
        try {
            // 生成 Excel 文件
            byte[] excelContent = excelService.generateExcelFile();
            
            // 设置响应头
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
            headers.setContentDispositionFormData("attachment", "report.xlsx");
            
            // 返回文件内容
            return new ResponseEntity<>(excelContent, headers, HttpStatus.OK);
        } catch (IOException e) {
            return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
}
```
```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.openxml4j.exceptions.InvalidFormatException;
import org.apache.poi.poifs.filesystem.POIFSFileSystem;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.ByteArrayOutputStream;
import java.util.List;
import java.util.ArrayList;
public class ExcelService {
    
    public void processExcelFile(File file) throws IOException, InvalidFormatException {
        try (InputStream fis = new FileInputStream(file)) {
            // 根据文件后缀自动确定使用 XSSFWorkbook 或 HSSFWorkbook
            Workbook workbook = WorkbookFactory.create(fis);
            
            // 获取第一个工作表
            Sheet sheet = workbook.getSheetAt(0);
            
            // 遍历所有行
            for (Row row : sheet) {
                // 遍历所有单元格
                for (Cell cell : row) {
                    switch (cell.getCellType()) {
                        case STRING:
                            System.out.print(cell.getStringCellValue() + "\t");
                            break;
                        case NUMERIC:
                            if (DateUtil.isCellDateFormatted(cell)) {
                                System.out.print(cell.getDateCellValue() + "\t");
                            } else {
                                System.out.print(cell.getNumericCellValue() + "\t");
                            }
                            break;
                        case BOOLEAN:
                            System.out.print(cell.getBooleanCellValue() + "\t");
                            break;
                        case FORMULA:
                            System.out.print(cell.getCellFormula() + "\t");
                            break;
                        default:
                            System.out.print("UNKNOWN TYPE\t");
                            break;
                    }
                }
                System.out.println();
            }
            
            // 关闭工作簿
            workbook.close();
        }
    }
    
    public byte[] generateExcelFile() throws IOException {
        // 创建一个新的工作簿
        Workbook workbook = new XSSFWorkbook();
        
        // 创建一个工作表
        Sheet sheet = workbook.createSheet("Report");
        
        // 创建表头
        Row headerRow = sheet.createRow(0);
        headerRow.createCell(0).setCellValue("ID");
        headerRow.createCell(1).setCellValue("Name");
        headerRow.createCell(2).setCellValue("Value");
        
        // 创建数据行
        for (int i = 1; i <= 5; i++) {
            Row row = sheet.createRow(i);
            row.createCell(0).setCellValue(i);
            row.createCell(1).setCellValue("Item " + i);
            row.createCell(2).setCellValue(i * 100);
        }
        
        // 将工作簿写入字节数组
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        workbook.write(bos);
        byte[] excelContent = bos.toByteArray();
        
        // 关闭工作簿
        workbook.close();
        
        return excelContent;
    }
}
```
上述代码创建了一个完整的 Spring Boot 应用程序，它能够处理 Excel 文件的上传和下载，并执行基本的 Excel 文件操作。
## 高级功能
### 1. 使用模板创建 Excel 文件
您可以使用现有的 Excel 模板文件来创建新的 Excel 文件。以下是使用模板的示例：
```java
public byte[] generateExcelFromTemplate(String templatePath) throws IOException, InvalidFormatException {
    // 加载模板文件
    InputStream templateStream = getClass().getResourceAsStream(templatePath);
    Workbook workbook = WorkbookFactory.create(templateStream);
    
    // 获取第一个工作表
    Sheet sheet = workbook.getSheetAt(0);
    
    // 修改模板内容
    Row row = sheet.getRow(0);
    if (row != null) {
        Cell cell = row.getCell(0);
        if (cell != null) {
            cell.setCellValue("New Value");
        }
    }
    
    // 将工作簿写入字节数组
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
    workbook.write(bos);
    byte[] excelContent = bos.toByteArray();
    
    // 关闭工作簿
    workbook.close();
    
    return excelContent;
}
```
### 2. 处理大型 Excel 文件
对于大型 Excel 文件，使用默认的方法可能会导致内存不足。为了避免这种情况，可以使用流式处理或分块处理：
```java
public void processLargeExcelFile(File file) throws IOException, InvalidFormatException {
    try (InputStream fis = new FileInputStream(file)) {
        // 创建工作簿，不加载所有数据到内存
        Workbook workbook = new XSSFWorkbook(fis);
        
        // 获取第一个工作表
        Sheet sheet = workbook.getSheetAt(0);
        
        // 使用物理行而不是逻辑行
        int firstRow = sheet.getFirstRowNum();
        int lastRow = sheet.getLastRowNum();
        
        for (int i = firstRow; i <= lastRow; i++) {
            Row row = sheet.getRow(i);
            if (row != null) {
                int firstCell = row.getFirstCellNum();
                int lastCell = row.getLastCellNum();
                
                for (int j = firstCell; j <= lastCell; j++) {
                    Cell cell = row.getCell(j);
                    if (cell != null) {
                        switch (cell.getCellType()) {
                            case STRING:
                                System.out.print(cell.getStringCellValue() + "\t");
                                break;
                            case NUMERIC:
                                if (DateUtil.isCellDateFormatted(cell)) {
                                    System.out.print(cell.getDateCellValue() + "\t");
                                } else {
                                    System.out.print(cell.getNumericCellValue() + "\t");
                                }
                                break;
                            case BOOLEAN:
                                System.out.print(cell.getBooleanCellValue() + "\t");
                                break;
                            case FORMULA:
                                System.out.print(cell.getCellFormula() + "\t");
                                break;
                            default:
                                System.out.print("UNKNOWN TYPE\t");
                                break;
                        }
                    }
                }
                System.out.println();
            }
        }
        
        // 关闭工作簿
        workbook.close();
    }
}
```
### 3. 使用 POI 的单元格样式
Apache POI 允许您为 Excel 文件中的单元格设置样式。以下是设置单元格样式的示例：
```java
public void createExcelWithStyles() throws IOException {
    // 创建一个新的工作簿
    Workbook workbook = new XSSFWorkbook();
    
    // 创建一个工作表
    Sheet sheet = workbook.createSheet("Styles");
    
    // 创建行和单元格
    Row row = sheet.createRow(0);
    Cell cell = row.createCell(0);
    cell.setCellValue("Styled Cell");
    
    // 创建样式
   CellStyle style = workbook.createCellStyle();
    
    // 设置背景颜色
    style.setFillForegroundColor(IndexedColors.BLUE.getIndex());
    style.setFillPattern(FillPatternType.SOLID_FOREGROUND);
    
    // 设置边框
    style.setBorderBottom(BorderStyle.THIN);
    style.setBorderTop(BorderStyle.THIN);
    style.setBorderLeft(BorderStyle.THIN);
    style.setBorderRight(BorderStyle.THIN);
    
    // 设置字体
    Font font = workbook.createFont();
    font.setColor(IndexedColors.WHITE.getIndex());
    font.setBold(true);
    font.setFontHeightInPoints((short)14);
    
    style.setFont(font);
    
    // 应用样式到单元格
    cell.setCellStyle(style);
    
    // 将工作簿写入文件
    try (FileOutputStream fileOut = new FileOutputStream("styles.xlsx")) {
        workbook.write(fileOut);
    }
    
    // 关闭工作簿
    workbook.close();
}
```
### 4. 使用 POI 的图表功能
Apache POI 还支持在 Excel 文件中创建图表。以下是创建图表的示例：
```java
public void createExcelWithChart() throws IOException, InvalidFormatException {
    // 创建一个新的工作簿
    Workbook workbook = new XSSFWorkbook();
    
    // 创建一个工作表
    Sheet sheet = workbook.createSheet("Chart");
    
    // 创建数据
    int numRows = 5;
    int numCols = 2;
    
    for (int i = 0; i < numRows; i++) {
        Row row = sheet.createRow(i);
        row.createCell(0).setCellValue("Category " + (i+1));
        row.createCell(1).setCellValue((i+1)*100);
    }
    
    // 创建图表
    Drawing<?> drawing = sheet.createDrawingPatriarch();
    ClientAnchor anchor = drawing.createAnchor(0, 0, 0, 0, 0, 0, 6, 3);
    
    Chart<?> chart = drawing.createChart(anchor);
    
    // 添加数据系列
    ChartData<?> chartData = chart.getChartDataFactory().create();
    
    for (int i = 0; i < numCols; i++) {
        ChartDataSource<String> categoryAxis = chartData.fromColumn(0, 0, numRows-1, i == 0);
        ChartDataSource<Number> valueAxis = chartData.fromColumn(1, 0, numRows-1, i == 0);
        
        chartData.addSeries(categoryAxis, valueAxis);
    }
    
    chart.setChartData(chartData);
    
    // 将工作簿写入文件
    try (FileOutputStream fileOut = new FileOutputStream("chart.xlsx")) {
        workbook.write(fileOut);
    }
    
    // 关闭工作簿
    workbook.close();
}
```
