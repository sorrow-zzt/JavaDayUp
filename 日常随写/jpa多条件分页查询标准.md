## jpa多条件分页查询标准

使用jpa进行简单的分页查询时统一使用如下方式:

实例:

![1561346121989](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1561346121989.png)

![1561346128556](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1561346128556.png)

代码示例:

1. controller层进行数据传递和结果集封装

   ```java
   @RequestMapping(value="/queryByPage" , method = RequestMethod.POST)//分页请求
       public Result<List<SbCommand>> findAllByPage(@RequestBody SearchVoPage searchVoPage){
       	//调用service
           Page<SbCommand> list = sbCommandService.findAllByPage(searchVoPage);
           //获取分页对象的数据list,并用Result封装
           List<SbCommand> data = list.getContent();
           return new ResultUtil<List<SbCommand>>().setDataList(data, searchVoPage.getPageVo().getLimit(), list.getTotalElements());
       }
   ```

2. service层进行查询对象Example的创建和PageVo的初始化

   ```java
   public Page<SbCommand> findAllByPage(SearchVoPage searchVoPage) {
           SbCommand sbCommand = new SbCommand();
           sbCommand.setIp(searchVoPage.getSearchVo().getQueryText());
       //创建匹配器，即如何使用查询条件
       //全部模糊查询，即%{ip}%,忽略查询条件id,因为雪花算法生成id影响查询
           ExampleMatcher exampleMatcher = ExampleMatcher.matching().withMatcher("ip",ExampleMatcher.GenericPropertyMatchers.contains()).withIgnorePaths("id");
       //将匹配对象封装成Example对象,并初始化了pageVo,默认排序字段是creationtime,order是desc,如需更改:pageable = PageRequest.of(1, 10,new Sort(Sort.Direction.ASC,"xxx"));
           Example<SbCommand> example = Example.of(sbCommand,exampleMatcher);
           Page<SbCommand> sbCommands = sbCommandDao.findAll( example, PageUtil.initPage(searchVoPage.getPageVo()));
           return sbCommands;
       }
   ```

   3.dao:没有,只要dao extends JpaRepository就可以使用了,这些方法主要在两个接口中定义，一是QueryByExampleExecutor，一个是JpaRepository.