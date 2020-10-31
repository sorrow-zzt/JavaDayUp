# Hutool使用文档

### 1.Convert---类型转换

![1568856008109](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1568856008109.png)

#### 1.转为字符串

```java
//数字转字符串
int a = 1;
String aStr = Convert.toStr(a);//aStr为"1"

//数组转字符串
long[] b = {1,2,3,4,5};
String bStr = Convert.toStr(b);//bStr为："[1, 2, 3, 4, 5]"

```

#### 2.转为指定类型数组

```java
//字符串数组转整数型数组
String[] b = { "1", "2", "3", "4" };
Integer[] intArray = Convert.toIntArray(b);//结果为Integer数组

//long型数组转Integer数组
long[] c = {1,2,3,4,5};
Integer[] intArray2 = Convert.toIntArray(c);//结果为Integer数组
```

#### 3.转为日期对象

```java
String a = "2017-05-06";
Date value = Convert.toDate(a);//字符串转日期对象
```

#### 4.数组转为集合

```java
//任意数组转换为集合
Object[] aa = {"a", "你", "好", "", 1};
List<?> objects = Convert.toList(aa);
```

#### 5.16进制转为字符串

```java
String a = "我是一个小小的可爱的字符串";//toHex方法同样支持传入byte[]
//结果："e68891e698afe4b880e4b8aae5b08fe5b08fe79a848fafe788b1e79a84e5ad97e7aca6e4b8b2"
String hex = Convert.toHex(a, CharsetUtil.CHARSET_UTF_8);

String hex = "e68891e698afe4b880e4b8aae5b08fe5b08fe79a84e58fafe788b1e79a84e5ad97e7aca6e4b8b2";
//同样也可以使用hexToBytes方法将16进制转为byte[]
String raw = Convert.hexToStr(hex, CharsetUtil.CHARSET_UTF_8);
byte[] bytes = Convert.hexToBytes(hex);
```

#### 6.Unicode和字符串转换

```java
String a = "我是一个小小的可爱的字符串";

//结果为："\\u6211\\u662f\\u4e00\\u4e2a\\u5c0f\\u5c0f\\u7684\\u53ef\\u7231\\u7684\\u5b57\\u7b26\\u4e32"    
String unicode = Convert.strToUnicode(a);//字符串转Unicode

//结果为："我是一个小小的可爱的字符串"
String raw = Convert.unicodeToStr(unicode);//Unicode转字符串

```

#### 7.编码转换

```java
String a = "我不是乱码";
//转换后result为乱码
String result = Convert.convertCharset(a, CharsetUtil.UTF_8, CharsetUtil.ISO_8859_1);
String raw = Convert.convertCharset(result, CharsetUtil.ISO_8859_1, "UTF-8");
//Assert.assertEquals(raw, a);
```

#### 8.金额大小写转换

```java
double a = 67556.32;

//结果为："陆万柒仟伍佰伍拾陆元叁角贰分"
String digitUppercase = Convert.digitToChinese(a);
//注意 转换为大写只能精确到分（小数点儿后两位），之后的数字会被忽略。
```

#### 9.原始类和包装类转换

```java 
//去包装
Class<?> wrapClass = Integer.class;
//结果为：int.class
Class<?> unWraped = Convert.unWrap(wrapClass);

//包装
Class<?> primitiveClass = long.class;
//结果为：Long.class
Class<?> wraped = Convert.wrap(primitiveClass);
```

### 2.DateUtil---日期时间类

#### 1.Date、long、Calendar之间的相互转换

```java
//当前时间
Date date = DateUtil.date();
//当前时间,Calendar转Date
Date date2 = DateUtil.date(Calendar.getInstance());
//当前时间,时间戳转Date
Date date3 = DateUtil.date(System.currentTimeMillis());
//当前时间字符串，格式：yyyy-MM-dd HH:mm:ss
String now = DateUtil.now();
//当前日期字符串，格式：yyyy-MM-dd
String today= DateUtil.today();
```

#### 2.字符串转日期

```java
//yyyy-MM-dd HH:mm:ss
//yyyy-MM-dd
//HH:mm:ss
//yyyy-MM-dd HH:mm
//yyyy-MM-dd HH:mm:ss.SSS
String dateStr = "2017-03-01";//动识别格式转换
Date date = DateUtil.parse(dateStr);
```

#### 3.格式化日期输出

```java
String dateStr = "2017-03-01";
Date date = DateUtil.parse(dateStr);
Date date = DateUtil.parse(dateStr, "yyyy-MM-dd");

//结果 2017/03/01
String format = DateUtil.format(date, "yyyy/MM/dd");

//常用格式的格式化，结果：2017-03-01
String formatDate = DateUtil.formatDate(date);

//结果：2017-03-01 00:00:00
String formatDateTime = DateUtil.formatDateTime(date);

//结果：00:00:00
String formatTime = DateUtil.formatTime(date);
```

#### 4.获取年月日

```java
//获得年的部分
int year = DateUtil.year(date);
=====================================================================================
System.out.println(DateUtil.now());//2019-09-19 11:28:52
System.out.println(DateUtil.year(new Date()));//2019
System.out.println(DateUtil.month(new Date()));//8,从0开始,实际要+1
System.out.println(DateUtil.dayOfMonth(new Date()));//19
System.out.println(DateUtil.dayOfWeek(new Date()));//5,从周六开始,实际要-1
```

#### 5.每天的开始和结束时间

```java
String dateStr = "2017-03-01 22:33:23";
Date date = DateUtil.parse(dateStr);

//一天的开始，结果：2017-03-01 00:00:00
Date beginOfDay = DateUtil.beginOfDay(date);

//一天的结束，结果：2017-03-01 23:59:59
Date endOfDay = DateUtil.endOfDay(date);

```

#### 6.日期偏移

```java
String dateStr = "2017-03-01 22:33:23";
Date date = DateUtil.parse(dateStr);

//结果：2017-05-01 22:33:23	//加2个月,适用所有偏移
Date newDate = DateUtil.offset(date, DateField.MONTH, 2);
//常用偏移，结果：2017-03-04 22:33:23	加3天
DateTime newDate2 = DateUtil.offsetDay(date, 3);
//常用偏移，结果：2017-03-01 19:33:23	减3小时
DateTime newDate3 = DateUtil.offsetHour(date, -3);

//昨天
DateUtil.yesterday()
//明天
DateUtil.tomorrow()
//上周
DateUtil.lastWeek()
//下周
DateUtil.nextWeek()
//上个月
DateUtil.lastMonth()
//下个月
DateUtil.nextMonth()
```

#### 7.日期时间差

```java
String dateStr1 = "2017-05-01 22:37:26";
Date date1 = DateUtil.parse(dateStr1);
String dateStr2 = "2017-04-01 23:33:23";
Date date2 = DateUtil.parse(dateStr2);

//相差一个月，31天
long betweenDay = DateUtil.between(date1, date2, DateUnit.DAY);
=================================================================================
//Level.MINUTE表示精确到分,SECOND表示精确到秒
String formatBetween = DateUtil.formatBetween(date1,date2, BetweenFormater.Level.SECOND);
//输出：29天23小时4分3秒
Console.log(formatBetween);
```

#### 8.年龄和闰年

```java
//年龄
int age = DateUtil.ageOfNow("1990-01-30");
System.out.println(age);//29
//是否闰年
boolean leapYear = DateUtil.isLeapYear(2017);
System.out.println(leapYear);//false
```

### 3.StrUtil---字符串工具类

#### 1.判断是否为空字符串

```java
//hasEmpty==isEmpty,hasBlank==isBlank,isBlank可以判断不可见字符
System.out.println(StrUtil.hasEmpty(""));//true
System.out.println(StrUtil.hasEmpty(null));//true
System.out.println(StrUtil.isEmpty("  "));//false
System.out.println(StrUtil.isBlank(""));//true
System.out.println(StrUtil.isBlank(null));//true
System.out.println(StrUtil.hasBlank("  "));//true
```

#### 2.去除字符串的前后缀

```java
StrUtil.removeSuffix("a.jpg", ".jpg");
StrUtil.removePrefix("a.jpg", "a.");
```

#### 3.占位符方法

```java
String template = "{}爱{}，就像老鼠爱大米";
String str = StrUtil.format(template, "我", "你"); //str -> 我爱你，就像老鼠爱大米
```

### 4.NumberUtil---数字工具类

#### 1.double类型的四则运算

```java
double n1 = 1.234;
double n2 = 1.234;
double result;
//对float、double、BigDecimal做加减乘除操作
result = NumberUtil.add(n1, n2);
result = NumberUtil.sub(n1, n2);
result = NumberUtil.mul(n1, n2);
result = NumberUtil.div(n1, n2);
```

#### 2.保留小数

```java
double te1=123456.123456;
double te2=123456.128456;
Console.log(round(te1,4));//结果:123456.1235
Console.log(round(te2,4));//结果:123456.1385
```

#### 3.随机数

```java
int[] ints = NumberUtil.generateRandomNumber(1, 10, 3);
System.out.println(Arrays.toString(ints));//1,4,8
Integer[] ints = NumberUtil.generateBySet(1, 10, 3);
System.out.println(Arrays.toString(ints));//2,5,7
```

#### 4.判断数字类型

```java
String n3 = "1.234";
//判断是否为数字、整数、浮点数
NumberUtil.isNumber(n3);
NumberUtil.isInteger(n3);
NumberUtil.isDouble(n3);
```

### 5.HttpUtil---Http客户端工具类

#### 1.Get请求

```java
// 最简单的HTTP请求，可以自动通过header等信息判断编码，不区分HTTP和HTTPS
String result1= HttpUtil.get("https://www.baidu.com");

// 当无法识别页面编码的时候，可以自定义请求页面的编码
String result2= HttpUtil.get("https://www.baidu.com", CharsetUtil.CHARSET_UTF_8);

//可以单独传入http参数，这样参数会自动做URL编码，拼接在URL中
HashMap<String, Object> paramMap = new HashMap<>();
paramMap.put("city", "北京");

String result3= HttpUtil.get("https://www.baidu.com", paramMap);
```

#### 2.POST请求

```java
HashMap<String, Object> paramMap = new HashMap<>();
paramMap.put("city", "北京");

String result= HttpUtil.post("https://www.baidu.com", paramMap);
```

#### 3.文件上传

```java
HashMap<String, Object> paramMap = new HashMap<>();
//文件上传只需将参数中的键指定（默认file），值设为文件对象即可，对于使用者来说，文件上传与普通表单提交并无区别
paramMap.put("file", FileUtil.file("D:\\face.jpg"));

String result= HttpUtil.post("https://www.baidu.com", paramMap);
```

#### 4.文件下载

```java
String fileUrl = "http://mirrors.sohu.com/centos/7.3.1611/isos/x86_64/CentOS-7-x86_64-DVD-1611.iso";

//将文件下载后保存在E盘，返回结果为下载文件大小
long size = HttpUtil.downloadFile(fileUrl, FileUtil.file("e:/"));
System.out.println("Download size: " + size);
```

#### 5.文件下载感知进度

```java
//带进度显示的文件下载
HttpUtil.downloadFile(fileUrl, FileUtil.file("e:/"), new StreamProgress(){

    @Override
    public void start() {
        Console.log("开始下载。。。。");
    }

    @Override
    public void progress(long progressSize) {
        Console.log("已下载：{}", FileUtil.readableFileSize(progressSize));
    }

    @Override
    public void finish() {
        Console.log("下载完成！");
    }
});
```

### 6.ExcelUtil---Excel工具

#### 1.读取Excel

```java
//1.读取Excel中所有行和列，都用列表表示
ExcelReader reader = ExcelUtil.getReader("d:/aaa.xlsx");
List<List<Object>> readAll = reader.read();

//2.读取为Map列表，默认第一行为标题行，Map中的key为标题，value为标题对应的单元格值。
ExcelReader reader = ExcelUtil.getReader("d:/aaa.xlsx");
List<Map<String,Object>> readAll = reader.readAll();

//3.读取为Bean列表，Bean中的字段名为标题，字段值为标题对应的单元格值。
ExcelReader reader = ExcelUtil.getReader("d:/aaa.xlsx");
List<Person> all = reader.readAll(Person.class);

```

#### 2.自定义工具上传Excel

```java
public class ExcelUtils {
    public static List<Map<String,Object>> readExcel2List(String path){
        ExcelReader reader = ExcelUtil.getReader(path);
        return reader.readAll();
    }

    public static ResponseEntity<byte[]> downLoadExcel(String path){
        File file=new File(path);
        HttpHeaders headers = new HttpHeaders();// 设置一个head
        try {
            headers.setContentDispositionFormData("attachment", URLEncoder.encode(file.getName(),"UTF-8"));// 文件的属性，也就是文件叫什么吧
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);// 内容是字节流
        return new ResponseEntity<>(FileUtil.readBytes(file), headers, HttpStatus.OK);// 开始下载
    }
}
==============================================================================
	List<Map<String, Object>> data = ExcelUtils.readExcel2List(path);
	for (Map<String, Object> map : data) {
        SbSlm sbSlm = new SbSlm();
        String imei = (String) map.get("IMEI");
        String modelid = (String) map.get("型号");
        String code2 = (String) map.get("二维码");
        String serial = (String) map.get("编号");
    }
```

#### 3.自定义工具下载Excel

```java
		//代码实现下载Excel,工具类自动处理好Excel格式		
		ExcelData data = new ExcelData();
        data.setName("终端信息"+ DateUtil.format(new Date(),"yyyy-MM-dd"));
        List<String> titles = new ArrayList<String>();
        titles.add("型号");
        titles.add("IMEI");
        titles.add("编号");
        titles.add("二维码");
        data.setTitles(titles);

        List<List<Object>> rows = new ArrayList<List<Object>>();
        for(int i = 0, length = list.size();i<length;i++){
                List<Object> row = new ArrayList<Object>();
                row.add(list.get(i).getModelid());
                row.add(list.get(i).getUid());
                row.add(list.get(i).getSerial());
                row.add(list.get(i).getCode2());
                rows.add(row);
        }
        data.setRows(rows);
        try{
            com.cosmos.common.util.ExcelUtils.exportExcel(response,"终端信息"+ DateUtil.format(new Date(),"yyyy-MM-dd"),data);
        }catch (Exception e){
            e.printStackTrace();
        }
====================================================================================
public class ExcelData implements Serializable {
	private static final long serialVersionUID = 1L;
	/**
     * 表头
     */
    private List<String> titles; 
    /**
     * 数据
     */
    private List<List<Object>> rows; 
    /**
     * 页签名称
     */
    private String name;
 	@Getter
    @Setter
}
==================================================================================
public class ExcelUtils {
 
    /**
     * 使用浏览器选择路径下载
     * @param response
     * @param fileName
     * @param data
     * @throws Exception
     */
    public static void exportExcel(HttpServletResponse response, String fileName, ExcelData data) throws Exception {
        // 告诉浏览器用什么软件可以打开此文件
        response.setHeader("content-Type", "application/vnd.ms-excel");
        // 下载文件的默认名称
        response.setHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(fileName + ".xlsx", "utf-8"));
        exportExcel(data, response.getOutputStream());
    }
 
    public static int generateExcel(ExcelData excelData, String path) throws Exception {
        File f = new File(path);
        FileOutputStream out = new FileOutputStream(f);
        return exportExcel(excelData, out);
    }
 
    private static int exportExcel(ExcelData data, OutputStream out) throws Exception {
        XSSFWorkbook wb = new XSSFWorkbook();
        int rowIndex = 0;
        try {
            String sheetName = data.getName();
            if (null == sheetName) {
                sheetName = "Sheet1";
            }
            XSSFSheet sheet = wb.createSheet(sheetName);
            rowIndex = writeExcel(wb, sheet, data);
            wb.write(out);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //此处需要关闭 wb 变量
            out.close();
        }
        return rowIndex;
    }
 
    /**
     * 表不显示字段
     * @param wb
     * @param sheet
     * @param data
     * @return
     */
//    private static int writeExcel(XSSFWorkbook wb, Sheet sheet, ExcelData data) {
//        int rowIndex = 0;
//        writeTitlesToExcel(wb, sheet, data.getTitles());
//        rowIndex = writeRowsToExcel(wb, sheet, data.getRows(), rowIndex);
//        autoSizeColumns(sheet, data.getTitles().size() + 1);
//        return rowIndex;
//    }
 
    /**
     * 表显示字段
     * @param wb
     * @param sheet
     * @param data
     * @return
     */
    private static int writeExcel(XSSFWorkbook wb, Sheet sheet, ExcelData data) {
        int rowIndex = 0;
        rowIndex = writeTitlesToExcel(wb, sheet, data.getTitles());
        rowIndex = writeRowsToExcel(wb, sheet, data.getRows(), rowIndex);
        autoSizeColumns(sheet, data.getTitles().size() + 1);
        return rowIndex;
    }
    /**
     * 设置表头
     *
     * @param wb
     * @param sheet
     * @param titles
     * @return
     */
    private static int writeTitlesToExcel(XSSFWorkbook wb, Sheet sheet, List<String> titles) {
        int rowIndex = 0;
        int colIndex = 0;
        Font titleFont = wb.createFont();
        //设置字体
        titleFont.setFontName("simsun");
        //设置粗体
       // titleFont.setBoldweight(Short.MAX_VALUE);
        //设置字号
        titleFont.setFontHeightInPoints((short) 14);
        //设置颜色
        titleFont.setColor(IndexedColors.BLACK.index);
        XSSFCellStyle titleStyle = wb.createCellStyle();
        //水平居中
        titleStyle.setAlignment(HorizontalAlignment.CENTER);
        //垂直居中
        titleStyle.setVerticalAlignment(VerticalAlignment.CENTER);
        //设置图案颜色
        titleStyle.setFillForegroundColor(new XSSFColor(new Color(182, 184, 192)));
        //设置图案样式
        titleStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);
        titleStyle.setFont(titleFont);
        setBorder(titleStyle, BorderStyle.THIN, new XSSFColor(new Color(0, 0, 0)));
        Row titleRow = sheet.createRow(rowIndex);
        titleRow.setHeightInPoints(25);
        colIndex = 0;
        for (String field : titles) {
            Cell cell = titleRow.createCell(colIndex);
            cell.setCellValue(field);
            cell.setCellStyle(titleStyle);
            colIndex++;
        }
        rowIndex++;
        return rowIndex;
    }
 
    /**
     * 设置内容
     *
     * @param wb
     * @param sheet
     * @param rows
     * @param rowIndex
     * @return
     */
    private static int writeRowsToExcel(XSSFWorkbook wb, Sheet sheet, 

List<List<Object>> rows, int rowIndex) {
        int colIndex;
        Font dataFont = wb.createFont();
        dataFont.setFontName("simsun");
        dataFont.setFontHeightInPoints((short) 14);
        dataFont.setColor(IndexedColors.BLACK.index);
 
        XSSFCellStyle dataStyle = wb.createCellStyle();
        dataStyle.setAlignment(HorizontalAlignment.CENTER);
        dataStyle.setVerticalAlignment(VerticalAlignment.CENTER);
        dataStyle.setFont(dataFont);
        setBorder(dataStyle, BorderStyle.THIN, new XSSFColor(new Color(0, 0, 0)));
        for (List<Object> rowData : rows) {
            Row dataRow = sheet.createRow(rowIndex);
            dataRow.setHeightInPoints(25);
            colIndex = 0;
            for (Object cellData : rowData) {
                Cell cell = dataRow.createCell(colIndex);
                if (cellData != null) {
                    cell.setCellValue(cellData.toString());
                } else {
                    cell.setCellValue("");
                }
                cell.setCellStyle(dataStyle);
                colIndex++;
            }
            rowIndex++;
        }
        return rowIndex;
    }
 
    /**
     * 自动调整列宽
     *
     * @param sheet
     * @param columnNumber
     */
    private static void autoSizeColumns(Sheet sheet, int columnNumber) {
        for (int i = 0; i < columnNumber; i++) {
            int orgWidth = sheet.getColumnWidth(i);
            sheet.autoSizeColumn(i, true);
            int newWidth = (int) (sheet.getColumnWidth(i) + 100);
            if (newWidth > orgWidth) {
                sheet.setColumnWidth(i, newWidth);
            } else {
                sheet.setColumnWidth(i, orgWidth);
            }
        }
    }
 
    /**
     * 设置边框
     *
     * @param style
     * @param border
     * @param color
     */
    private static void setBorder(XSSFCellStyle style, BorderStyle border, XSSFColor color) {
        style.setBorderTop(border);
        style.setBorderLeft(border);
        style.setBorderRight(border);
        style.setBorderBottom(border);
        style.setBorderColor(BorderSide.TOP, color);
        style.setBorderColor(BorderSide.LEFT, color);
        style.setBorderColor(BorderSide.RIGHT, color);
        style.setBorderColor(BorderSide.BOTTOM, color);
    }
}
```

