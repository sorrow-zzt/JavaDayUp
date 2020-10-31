## Specification用法简介

![1572589309031](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1572589309031.png)

#### 1.直接看代码

```java
	@Override
	public Page<Ledprogram> findByNameAndTypePage(SearchVoPage searchVoPage) {
		/**多条件数据查询???**/
		@SuppressWarnings("serial")
		Specification<Ledprogram> spec = new Specification<Ledprogram>() {
			@Override
			public Predicate toPredicate(Root<Ledprogram> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
				Path<String> namePath=root.get("name");
				Path<String> typePath = root.get("type");
				Path<Date> creationtime = root.get("creationtime");
				List<Predicate> list =new ArrayList<>();
				if(StringUtils.isNotBlank(searchVoPage.getSearchVo().getQueryText())&&"1".equals(searchVoPage.getSearchVo().getQueryText())) {
					CriteriaBuilder.In<String> in = cb.in(typePath);
					for (String img : imgs) {
						in.value(img);
					}
					list.add(in);
				}else if(StringUtils.isNotBlank(searchVoPage.getSearchVo().getQueryText())&&"2".equals(searchVoPage.getSearchVo().getQueryText())){
					CriteriaBuilder.In<String> in = cb.in(typePath);
					for (String video : videos) {
						in.value(video);
					}
					list.add(in);
				}
				if(StringUtils.isNotBlank(searchVoPage.getSearchVo().getQueryText2())) {
					list.add(cb.like(namePath, "%"+searchVoPage.getSearchVo().getQueryText2()+"%"));
				}
				if(ObjectUtil.isNotNull(searchVoPage.getSearchVo().getStartTime())&&ObjectUtil.isNotNull(searchVoPage.getSearchVo().getEndTime())){
					list.add(cb.between(creationtime,searchVoPage.getSearchVo().getStartTime(),searchVoPage.getSearchVo().getEndTime()));
				}
				if(StrUtil.isNotEmpty(searchVoPage.getSearchVo().getStartDate())&&StrUtil.isNotEmpty(searchVoPage.getSearchVo().getEndDate())){
					list.add(cb.between(creationtime,new Date(Long.parseLong(searchVoPage.getSearchVo().getStartDate())),new Date(Long.parseLong(searchVoPage.getSearchVo().getEndDate()))));
				}
				Predicate[] p = new Predicate[list.size()];
				return cb.and(list.toArray(p));
			}
		};
		Page<Ledprogram> pageList =ledprogramDao.findAll(spec, PageUtil.initPage(searchVoPage.getPageVo()));
		return pageList;
	}
```

#### 2.看图分析

```java
	@GetMapping("/event/list")
    public ApiReturnObject findAllEvent(String event,Timestamp registerTime,Integer pageNumber,Integer pageSize) {
        if(pageNumber==null) pageNumber=1;
        if(pageSize==null) pageNumber=10;
        //分页
        //Pageable是接口，PageRequest是接口实现，new PageRequest()是旧方法，PageRequest.of()是新方法
        //PageRequest.of的对象构造函数有多个，page是页数，初始值是0，size是查询结果的条数，后两个参数参考Sort对象的构造方法
        Pageable pageable = PageRequest.of(pageNumber,pageSize,Sort.Direction.DESC,"id");
        //Specification查询构造器
        Specification<Event> specification=new Specification<Event>() {
            private static final long serialVersionUID = 1L;

            @Override
            public Predicate toPredicate(Root<Event> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder) {
                Predicate condition1 = null;
                if(StringUtils.isNotBlank(eventTitle)) {
                    condition1 = criteriaBuilder.like(root.get("eventTitle"),"%"+eventTitle+"%");
                }else {
                    condition1 = criteriaBuilder.like(root.get("eventTitle"),"%%");
                }
                Predicate condition2 = null;
                if(registerTime!=null) {
                    condition2 = criteriaBuilder.greaterThan(root.get("registerTime"), registerTime);
                }else {
                    condition2 = criteriaBuilder.greaterThan(root.get("registerTime"), new Timestamp(1514736000000L));
                }
                //Predicate conditionX=criteriaBuilder.and(condition1,condition2);
                //query.where(conditionX);
                query.where(condition1,condition2);
                //query.where(getPredicates(condition1,condition2)); //这里可以设置任意条查询条件
                return null;  //这种方式使用JPA的API设置了查询条件，所以不需要再返回查询条件Predicate给Spring Data Jpa，故最后return null
            }
        };
        Page<Event> list=eventRepository.findAll(specification, pageable);

        return ApiReturnUtil.page(list);
    }
```

##### 步骤1:

![1564987705980](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1564987705980.png)

```java
	@Override
	public Page<WebUser> findAll(WebUser webUser, Pageable pageable) {
		Page<WebUser> page = webUserRepository
				.findAll((Root<WebUser> root, CriteriaQuery<?> query, CriteriaBuilder cb) -> {
					List<Predicate> predicates = new ArrayList<Predicate>();
					predicates.add(cb.like(root.get("userName").as(String.class), "%"+webUser.getUserName() + "%"));
					predicates.add(cb.like(root.get("email").as(String.class), "%"+webUser.getEmail() + "%"));
					predicates.add(cb.like(root.get("phoneNo").as(String.class), "%"+webUser.getPhoneNo() + "%"));
					predicates.add(cb.equal(root.get("isValid").as(String.class), webUser.getIsValid()));
					
					SimpleDateFormat f = new SimpleDateFormat("yyyy-MM-dd");
					try {
						if (null != webUser.getCreateTimeStart() && !"".equals(webUser.getCreateTimeStart()))
							predicates.add(cb.greaterThanOrEqualTo(root.get("createTime").as(Date.class),
									f.parse(webUser.getCreateTimeStart())));
						if (null != webUser.getCreateTimeEnd() && !"".equals(webUser.getCreateTimeEnd()))
							predicates.add(cb.lessThan(root.get("createTime").as(Date.class),
									new Date(f.parse(webUser.getCreateTimeEnd()).getTime() + 24 * 3600 * 1000)));
					} catch (ParseException e) {
						e.printStackTrace();
					}
					return query.where(predicates.toArray(new Predicate[predicates.size()])).getRestriction();
				}, pageable);
		return page;
 
	}
```

##### 步骤2:

![1564987822068](C:\Users\lx-PC\AppData\Roaming\Typora\typora-user-images\1564987822068.png)

##### Criteria 查询 Demo:

```java
		//创建CriteriaBuilder安全查询工厂
        //CriteriaBuilder是一个工厂对象,安全查询的开始.用于构建JPA安全查询.
        CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
        //创建CriteriaQuery安全查询主语句
        //CriteriaQuery对象必须在实体类型或嵌入式类型上的Criteria 查询上起作用。
        CriteriaQuery<Item> query = criteriaBuilder.createQuery(Item.class);
        //Root 定义查询的From子句中能出现的类型
        Root<Item> itemRoot = query.from(Item.class);
        //Predicate 过滤条件 构建where字句可能的各种条件
        //这里用List存放多种查询条件,实现动态查询
        List<Predicate> predicatesList = new ArrayList<>();
        //name模糊查询 ,like语句
        if (name != null) {
            predicatesList.add(
                    criteriaBuilder.and(
                            criteriaBuilder.like(
                                    itemRoot.get(Item_.itemName), "%" + name + "%")));
        }
        // itemPrice 小于等于 <= 语句
        if (price != null) {
            predicatesList.add(
                    criteriaBuilder.and(
                            criteriaBuilder.le(
                                    itemRoot.get(Item_.itemPrice), price)));
        }
        //itemStock 大于等于 >= 语句
        if (stock != null) {
            predicatesList.add(
                    criteriaBuilder.and(
                            criteriaBuilder.ge(
                                    itemRoot.get(Item_.itemStock), stock)));
        }
        //where()拼接查询条件
        query.where(predicatesList.toArray(new Predicate[predicatesList.size()]));
        TypedQuery<Item> typedQuery = entityManager.createQuery(query);
        List<Item> resultList = typedQuery.getResultList();
        return resultList;
```

#####  Specification  查询 Demo:

```java
	public Page<Item> findByConditions(String name, Integer price, Integer stock, Pageable page) {
     Page<Item> page = itemRepository.findAll((root, criteriaQuery, criteriaBuilder) -> {
            List<Predicate> predicatesList = new ArrayList<>();
            //name模糊查询 ,like语句
            if (name != null) {
                predicatesList.add(
                        criteriaBuilder.and(
                                criteriaBuilder.like(
                                        root.get(Item_.itemName), "%" + name + "%")));
            }
            // itemPrice 小于等于 <= 语句
            if (price != null) {
                predicatesList.add(
                        criteriaBuilder.and(
                                criteriaBuilder.le(
                                        root.get(Item_.itemPrice), price)));
            }
            //itemStock 大于等于 >= 语句
            if (stock != null) {
                predicatesList.add(
                        criteriaBuilder.and(
                                criteriaBuilder.ge(
                                        root.get(Item_.itemStock), stock)));
            }
            return criteriaBuilder.and(
                    predicatesList.toArray(new Predicate[predicatesList.size()]));
        }, page);
    return page;
}
```

