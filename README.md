# Emall_shangguigu
 an e-mall project based on micro service structure，在b站搜到的尚硅谷谷粒商城项目，使用springboot和springcloud

### This is the full-stack development I'm currently working on. This Project focused on developing an E-mall like Taobao, Jingdong with the Micro-Service Structure using Java.I'll put the stage result to this file to share during the development.

[front_end_codes](https://github.com/Niyx52094/Emall_shangguigu/tree/main/renren-fast-vue_beforeP75)
[back_end_base_model_gulimall_code](https://github.com/Niyx52094/gulimall_base)

## 项目日志 Project log

### 2020.12.18

开始进行项目学习。

进行环境配置，使用vagrant搭建Linux环境虚拟机，安装Docker，在docker中安装Mysql5.7 和redis。

设置docker，将容器中的软件文件夹映射到docker外部的linux环境中，两者一体，哪个修改会影响另一个，便于查看和修改

redis持久化配置

配置maven环境，安装vscode，配置与码云Gitee的同步。

创建微服务项目，分为商品服务，仓储服务，订单服务，优惠卷服务，用户服务。微服务中每一个项目都可以单独进行数据库操作。



![Image text](https://github.com/fishingfreedom/figures/blob/main/QQ%E6%88%AA%E5%9B%BE20201219095252.png)


导入mysql五个表单。

### 2020.12.19 ,搭建后台管理系统


前端网关的admin-vue使用码云上的“人人开源”项目下的文件，分别为fast，fast-vue,generator,下载对应nodejs版本10.16.3。把fast-vue里面的.git删除后放入vscode。同样把.git删除后fast与generator放入到idea的proj中，Generator与fast中的resources下application.yml从本机地址修改为虚拟机地址。fast中启动，vscode同样启动，最后获得两者配套的网页(我想这应该就是前后端分离了吧)。

![Image_test](https://github.com/fishingfreedom/figures/blob/main/admin%E7%BD%91%E5%85%B3.png)

generator可以自动生成entity，dao,controller,service等，非常方便。但需要先启动，成功获得数据库表的内容才能进行逆向连接。网页如图，全选后生成代码即可。

![Image_test](https://github.com/fishingfreedom/figures/blob/main/generator%E7%BD%91%E5%85%B3.png)

使用Mybatis-plus进行CRUD操作，直接可以使用生成的服务类名后加.inset()/.delete()/.updatedById and so on。非常方便快捷。这样就不需要写XML

遇到的问题：
  1. nodejs安装后需要进行npm install，这一步如果不用淘宝镜像下载下来的东西会有很多错误，用fix也修复不了。
  2. genrator 的template慎重修改，我不小心注释了每个类前的@RequstMapping，结果生成的controller都没有这个，导致spring把这一整队controller变为一个，所以造成了很多方法冲突。。。改完之后增删改查成功。


### 2020.12.26

* 1.学习了基本的Mybais-plus的操作，首先要在当前方法下创建实体，然后用Dao后面加.insert(该实体), .updatedById(this entity), .selectOne(..) , .selectByMap(..),.deleteByid(..)等进行增删改查。

* 2. 配置springcloud
  *recently spring cloud has some problems about its components, so we use spring cloud alibaba.
  
  配置方案为：
    * Spring cloud Alibaba: Nacos: 注册中心（服务发现/注册）
    * Spring cloud Alibaba: Nacos: 配置中心（动态配置管理）
    * Spring cloud  Riboon：负载均衡
    * Spring cloud  openFeign: 声明式Http客户端（调用远程服务）
    * Spring cloud Alibaba Sentinel：服务容错（限流，降级，熔断）
    * Spring cloud  Gateway：API网关(webflux编程模式)
    * Spring cloud  Sleuth：调用链监控
    * Spring cloud Alibaba Seata：原Fescar，分布式事务解决方案

  ### 2.1 配置nacos注册中心
  
    注册中心类似于“滴滴”， 将服务统一管理并显示，由用户来订阅，并使用。服务只有注册过了才能显示，服务若下线，一定心跳时间后注册中心的服务也下线，用户无法使用
      
        2.1.1 添加依赖
        ```
        <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        ```
        2.1.2 在yml文件中加入自己nacos服务器路径和本服务的名字
        
        ```
        cloud:
          nacos:
            discovery:
              server-addr: 127.0.0.1:8848
        application:
          name: gulimall-coupon
         ```
         2.1.3 在启动类上添加服务发现
            ```
            @EnableDiscoveryClient
            ```
   ### 2.2 使用openFeign进行线上服务的交互
    
    openFeign能够使在注册中心注册过的服务互相调用，openFeign属于声明式的HTTP客户端，（需要创建接口）整合了Ribon负载均衡和Hystrix服务熔断
    
    案例：比如member服务要到优惠券服务中调用这个会员有几个优惠卷，先通过注册中心，发现优惠券服务在哪些机器，然后选择其中一个机器上的优惠券服务发送请求，优惠券服务进行响应。进行交互。
          
     2.2.1 先在coupon服务中，新建一个方法，获取一个coupon实体类，并加入到R（返回给前端的json封装），这作为一个远程服务，随后会被member调用。Requestmapping里面为“/member/list”
          
     2.2.2 首先引入open-feign，编写一个openfeign的接口，并告诉springcloud这个接口需要调用远程服务。所有的远程接口都放在“feign”文件夹下（放在想要调用其他服务的服务目录下）。声明接口的每一个方法都是调用哪个远程服务的名字（该服务的名字在nacos的注册中心里找到。放在接口方法上方的FeignClient那里），最后加完整路径名（“coupon/coupon/member/list”）。
          
     2.2.3 开启远程调用功能，在启动类上加@EnablFeignClients(basepackge="接口的绝对路径位置"),在里面把远程接口的包导入。这样这个启动类每次打开就能自动进行远程接口的扫描。
          
     2.2.4 随后在member服务中，有一个会员为张三，想要使用远程调用自己所有的优惠卷。则使用这个远程接口，并注入后，使用该远程接口可以获得coupon里面的相对应的方法，并将张三和优惠卷包装起来返回。
      
  ### 2.3 使用nacos作为配置中心
    
    首先引入相应的依赖，在resources文件夹下创建bootstrap.properties, 在该文件中放入此时的服务名字，以及nacos的路径（127.0.0.1：8848）. 使用 @Value进行属性注入。
   
   * 以前都是使用在application.properties 下面进行配置。然后再用```@value```进行属性注入，但这样每次一改的话服务都要停止在重新上线
   * 现在把配置文件放入配置中心的话就可以动态的改配置文件，服务也不需要停止。点击“+”新建，配置名字需要在微服务下找，这个coupon的微服务启动后，会出现下面代码，他会自动搜索“gulimall-coupon.properties”。因此我们创建的文件名也要叫 "gulimall-coupon.properties". 在该文件下配置完成后，还需要在Controller上加```@RefreshScope```自动刷新。当然```@value```进行属性注解仍然不能少。 ```@value(“$(配置项的名字)”)```。如果配置中心和当前应用的配置文件都配置了相同的项， 优先使用**配置中心的项**。
   
   * 学习了“命名空间” ， 对不同配置进行隔离，在不同环境下可以用不同配置。默认情况下有一个默认值 public，默认的新增所有配置都在public文件夹下。每个命名文件有一个独一无二的识别符，需要在 ```bootstrap.properties```下加入```spring.cloud.nacos.config.namespace=识别码```，就能定位到相应的命名空间
   
   * 在同一个命名空间中，还可以将配置进行分组，idea识别的话要在 ```bootstrap.properties```加入```spring.cloud.nacos.config.group=组名```。
   
   ## 本微服务项目使用不同微服务名作为命名空间。每个命名空间中放入三个不同的组，分别为开发dev，测试test，和生产prod。
   
   * 学习了进行多配置集操作
   
   我们可以将resources下所有的配置文件都放到配置中心中，包括```application.yml```中的东西，将有关datasource的放入datasource.yml，mybatis的配置返给mybatis,其他放入other.yml。让如之后，```application.yml```实际上就已经没什么用了。而我们要获得配置中心中刚配置好的文件，需要在bootstrap中加入：```...config.ext-config[0].(id,group,refresh)=datasource.yml,dev,true``` 等，表示导入dev组的datasource.yml，并且保持动态更新。其他同理
   
   
 ### 2.4 使用Gateway网关
   
   网关的作用：当某一个服务用不了时，切换到能用的服务。网关动态的帮我们路由到指定准确的服务。还可以进行监控，写在服务上的话就进行了重复开发，因此放在服务于client之间进行鉴权，限流，路由，日志输出等。(Route predicate filter)
   ![gateway](https://github.com/fishingfreedom/figures/blob/main/gateway.png)
   
   当请求到达，路由判断是否符合某个断言（predicate）规则，如果符合，经过一系列的filter，则把这个请求发送到指定地方。
   
   案例：创建一个maven called “gateway” 导入注册中心，配置中心，gateway依赖，在启动类中加入注解```@EnalbleDiscoveryClient```。 在```application.properties```中加入自己的服务名字以及nacos注册中心的网址（本机地址）。在```bootstrap.properties```中配置nacos配置中心的参数。
  * 网关的设置在application.yml中加入：
     ```
     spring:
        cloud:
          gateway:
            route://固定配置
              - id: query_route
                uri: https://www.baidu.com
                predicates:
                  - Query=url,baidu
      ```
      则在网页中输入```localhost:88/url=baidu```就能跳转到经过网关路由到百度页面。
      
      
      
### 2.5开始学习前端知识 Basic front-end knowledge and Vue framework
  
           
  * 1. let的特性：
      let是局部变量，只能声明一次，var可以声明无数次。
      var有变量提升，意思是var无论在哪里声明，都会被认为在开头声明，所以前面没声明也可以用某个var值。但是赋值只会在后面，而不会被提前
  * 2. const 是常量，声明时必须初始化
  * 3. 解构表达式，可以快速赋值
      ```let arr=[2,1,3]
          let [a,b,c]=arr
          console.log(a,b,c)//输出2，1，3
      ```
   * 4. 字符串模板，使用反引号``` ` ```，${}可以在里面插入想要插入的变量或方法，而在字符串中串联
        ```
        let name=batman
        let age=100
        let str=`i am ${name}, my age is ${age}`
        console.log(str)//return i am batman, my age is 100
        
        ```

   * 5. 函数的定义
        ```
        function fun(a,b){
          return a+b
        }
        ```
   * 6. 箭头函数
      ```
      //以前声明一个方法，
      //var function print(obj){
      //   console.log(obj)
      //
      //现在可以简写为：
      var print=obj=>console.log(obj);
      //测试调用
      print(100); //print传一个obj参数
      var sum3=(a,b)=>{
      c=a+b;
      return a+c;
       }//大括号内部为方法体，就是函数sum2内部的方法。
       console.log(sum3(10,13)//return 33
       ```
### 2021.01.01
  * 7. Map 和Reduce, 
    map是类似stream()和python的map，将箭头函数内的操作对所有列表内的值进行操作。
    Recuce是回调上一次操作的函数
    ```
      //数组中新增了map和reduce的方法
      //map():接受一个函数，将原数组中的所有元素用这个函数处理后放入新数组返回。
      let arr=["1","20","-5","4"]

      // arr=arr.map((item)=>{
      //     return item*2
      // })
      
      //reduce()为数组中每一个元素依次执行回调函数，不包括数组中被删除或未被赋值的元素
      //[2，40，-10，8]
      //arr.reduce(callback,[initialValue]),reduce模板
      /*callback:依次为
        1.previousValue 上一次调用回调返回的值，或者是提供的初始值（initialValue)
        2.currentValue(数组中当前被处理的元素)
        3.index（当前元素在数组中的索引）
        4.array（调用reduce的数组）
        arr.reduce(a,b)就分别为第一个和第二个，所以是previous 和current
      */
    let result=arr.reduce((a,b)=>{
        console.log("上一次处理后： "+a);
        console.log("当前正在处理："+b);
        return a + b;
    });

    ```
  * 8. Promise 
  
  在javascript中，所有代码都是单线程执行。由于这个缺陷，导致javascript的所有网络操作，浏览器事件，都必须异步执行。异步执行可以用回调函数实现，一旦有一连串的ajax请求a,b,c,d后面的请求依赖前面的请求结果，就需要层层嵌套，这种缩进和层层嵌套的方式，非常容易造成上下文代码混乱，我们不得不小心处理内层函数域外层函数的数据，一旦函数使用了上层函数的变量，这种混乱就会加剧……总之，这种层层嵌套的方式，着实增加了神经的紧张程度
  ```
    function get(url, data) {
            return new Promise((resolve, reject) => {
                $.ajax({
                    url: url,
                    data: data,
                    success: function (data) {
                        resolve(data);
                    },
                    error: function (err) {
                        reject(err);
                    }
                })

            });
        }

        //使用这个getfunction，无限then套娃
        get("mock/user.json")
            .then((data) => {
                console.log("用户查询成功", data)
                return get(`mock/user_score_${data.id}.json`);
            })
            .then((data) => {
                console.log("课程查询成功", data)
                return get(`mock/course_score_${data.id}.json`);
            })
            .then((data)=>{
                console.log("课程成绩查询成功",data)
            })
            .catch((err)=>{
                console.log("出现以常",err);
            });
  ```
  
 * 9. 模块化
    
 * 什么是模块化：
   * 模块化就是把代码拆分，方便重复利用，类似java的导包，
   * 模块主要功能：export 和 import
   * Export命令用于规定模块的对外接口
   * Import命令用于导入其他模块提供的功能

 * 10.VUE框架学习
    * 首先使用```npm init -y```初始化，然后```npm install vue```
    * 学习了声明式渲染```new Vue```，双向绑定```v-model```，```v-on:```事件处理，```v-test，v-html```，```v-bind```绑定连接和属性，
    * 事件修饰符，```keyup```，已经最后的条件```v-if```和循环```v-for```
    * 计算属性和侦听器，过滤器，组件化，生命周期等，不在赘述
    * 总结vue框架，
     * el:绑定元素
     * data: 封装数据
     * methods:封装方法
     
    * vue项目模板：必须要由```<template>```,```<script>```,```<script>```组成，
    <template>放前端文本，
    <script>放vue实例
    <style>放文本样式
    
* 11.ElementUI
  导入vue的main.js文件下
```
  import Vue from 'vue';
  import ElementUI from 'element-ui';
  import 'element-ui/lib/theme-chalk/index.css';
  import App from './App.vue';

  Vue.use(ElementUI);

  new Vue({
  el: '#app',
  render: h => h(App)
  });

```
### 2.6 Product Service API

接下来“第一阶段”基本都是前端技术。很好的锻炼的全栈能力。
The first stage is front-end.
  
#### 2.6.1 Administrator API of Product category (Tree_strcture administration)
* 1.创建页面的商品分类三层级的树状结构。
  
使用controller，service interface，impl，and Dao， use Mybaits-plus, using stream() syntax to filter the corresponding menu. Here the getChildren() is the helper method to get all the children of the menu.
  ```
    //1.查出所有分类//check all the product name
    List<CategoryEntity> categoryEntities = baseMapper.selectList(null);
    //2.组装成父子的树形结构//combine as tree structure
      //2.1找到所有的一级分类（parent_id是0）
    List<CategoryEntity> level1Menus = categoryEntities.stream().filter((categoryEntity) -> {
        return categoryEntity.getParentCid() == 0;
    }).map((menu)->{
       menu.setChildren(getChildren(menu,categoryEntities));
       return menu;
    }).sorted((menu1,menu2)->{
        return (menu1.getSort()==null?0:menu1.getSort())-(menu2.getSort()==null?0:menu2.getSort());
    }).collect(Collectors.toList());//一级菜单
    return level1Menus;
}

  ```
* 2.学习了后台管理系统，显示树状结构，这里面涉及到，把renren-vue绑定的renren-fast路径8080改为全部通关网关88端口，用网关的路由功能路由到8080端口。同样，通过网关在“分类维护”下面路由到到商品服务，并成功显示上述的树状结构。当然要做到这两点都需要在nacos注册中心注册。

* 3.完成树状结构的删除，修改，新增并学习了逻辑删除。逻辑删除是没有把数据库真正删除的操作。确定一个状态位，状态位为0说明这个元素被删除了，就不会显示在前端。这个状态位在实体类里面需要用```@TableLogic```注解掉，那么正常使用delete就可以，数据库不会删除，会改状态位。
* 4.完成了树状结构的拖拽效果。并给予条件当全部层级大于3时不能拖拽成功。完成拖拽的层级，父节点Id,当前节点排序位置的前端显示。
  
### 2021.1.3
* 5.完成拖拽效果的数据收集，拖拽效果的开关，优化拖拽效果，使用“批量保存”在最后一步才进行数据库交互；
* 6.使用elementUI 完成“批量删除”

* **Finish the Category administration APIs**

#### 2.6.2 Administrator APIs of Product Brand
* 1.完成管理页面的优化，使用elementUI中的开关组件把“显示状态”从数字变成开关，开表示"1",关表示"0"，自定义显示模板用template slot-scope。scope可以抓取当前位置需要显示的所有"行",“列”，“内部管理”等数据。
* 2.完成开关监听事件“change”，在开关状态改变时前端收集改变后信息，用来回传给后端进行数据库更新
* 3.完成把文件logo地址的文本显示改为“上传文件”组件。使用阿里云的“服务器签名重传”，用户先向服务器获取一组唯一密码，加上这个密码后用户直接把文件上传到阿里云。使用```spring-cloud-alicloud-oss```依赖进行上传。
* 4.进行前端表单校验(front-end dataform validation),使用elementUi里面的```validator```获取value进行if-else校验，用callback返回校验失败文本。
* 5.进行后端表单校验(back-end dataform validation)，使用JSR303校验规则。在需要检验的实体类属性上加上注解比如```@NotNull/@NotBlack```等。在controller上相应的方法里面加上相应的```@Valid```和```BindingResult```,进行if-else校验
* 6.取消```BindingResult```，使用springMVC提供的```@ControllerAdvice```编写实体类```GulimallExceptionControllerAdvice```进行统一的异常管理。用@ResponceBody封装异常数据返回给前端显示。
* 7.使用分组校验功能，有的属性在新增时不需要添加，但是修改时必须非空，这样可以用groups来选择。编写两个接口```AddGroups```和```UpdatesGroups```，把controller里头的```@Valid```改为```@Validated({XXX.class})```这样就能让属性在不同状态下进行不同的校验。

* **Finish the brand administration APIs**

### 2021.01.09
#### 2.6.3 Administrator APIs of Product Attribute Groups

从这里开始不再进行前端的撰写，而是使用“接口文档”获得前端的请求已经响应返回的数据封装，从接口文档中知道后端的接收与返回，专注于后端的编写。
* 1.创建前端common文件夹，抽取```Category.vue```的树状结构回显。在```attrGroup.vue```中导入组件获得在属性页面显示的树状结构。
* 2.使用子父组件信息传递功能，使用```this.$emit("functionName",data)```来泄露给父组件函数，父组件使用```v-on```或者```@```来使用它。获得在点击上述树状结构时，被点击的节点的node，data，用于在旁边显示节点所拥有的属性。
* 3.使用上述获得的子节点catelogID获得```pms_attr_group```的相关属性分组.并使用mybatis-plus完成模糊查询的查询功能APIs.
* 4. 完成新增APIs,在新增中使用使用级联选择器来选择树状架构的子节点，级联选择器发送的请求时一个catelogPath的从根到叶节点的数组，前端显示用props来正确找到显示的name。默认的label在这里的项目是没有的。
* 5.完成修改功能APIs，发现属性Id没有回显，需要回显一个上述的根节点到叶子节点的数组，但是这个在数据库里面没有，所以在实体类里面加上后用```@Tablefield（exist=false）```注释掉。在service里面使用循环或者递归获得一整个路径。然后回显。
* 6.加入mybatis-plus的分页插件，正确获取记录的条数，每页显示数目等。
* 7.对Product brand 新增关联categories的分类，因为一个品牌可能在手机中有业务，也可能买家电，手表等。相当于操作```brand_category_relation```这张表，完成这个Controller的增删改查APIs。
* 8.对category和brand进行修改时，上述的关联表一旦查询出有响应值也要同步修改。
* **Finish the Attributes groups administration APIs**
#### 2.6.4 Administrator APIs of Product Attributes

*在前面所有实体回显与数据库表不一致的地方都加了注解，这是不规范的，业内现在使用```VO(ValueObject)```来做。新建一个VO文件夹，创建一个VO类能够集成数据库对应的实体类。在这个VO实体类里面加入回显给前端的额外属性。*
* 1.在vo中心怎AttrVo新增```attrgroup_id```新增```pms_attr```这张表，并同时新增```attr_attrgroup```这张关联表。获得attr里面的```attrgroup_id ```和自己的```attr_id```，编写一个```relation```实体类，加入这两个值后用mybatis-plus的```.insert```函数新增。
```
@Override
public void saveAttr(AttrVo attrVo) {
    AttrEntity attrEntity = new AttrEntity();
    //在两个实体类属性名完全相同时可以拷贝,前面的值拷贝后面的值
    BeanUtils.copyProperties(attrVo,attrEntity);
    //1.基本保存：
    this.save(attrEntity);
    //2.保存关联关系
    Long attrGroupId = attrVo.getAttrGroupId();
    Long attrId = attrEntity.getAttrId();//用attrentity的ID
    AttrAttrgroupRelationEntity attrAttrgroupRelationEntity = new AttrAttrgroupRelationEntity();
    attrAttrgroupRelationEntity.setAttrId(attrId);
    attrAttrgroupRelationEntity.setAttrGroupId(attrGroupId);
    attrAttrgroupRelationDao.insert(attrAttrgroupRelationEntity);
}
```
* 2.查询APIs的接口文档中发现后端回显时多了```属性分组名称```和```分类名称```，所以在controller里面要查询```attrgroup```，```category```这两张表，而查询这两张表又要查询响应的relarion表。回显的实体类是新建一个```AttrResVo```，加入这两个属性。使用```BeanUtils.copyProperties(a,b)```来获得一般通用的参数（a赋值给相同属性名称的b）。使用page的```getRecords```来获得需要改变的```List<AttrEntity>```将每一个```AttrEntity```使用java的流函数```stream()```来转换为```AttrResVo```。
```
List<AttrEntity> records = page.getRecords();
        //从AttrEntity转换为AttrResVo
        List<AttrResVo> res = records.stream().map((obj) -> {
            AttrResVo attrResVo = new AttrResVo();
            BeanUtils.copyProperties(obj,attrResVo);
            Long catelogId = obj.getCatelogId();
            //使用 pms_category 的 Dao获得name
            CategoryEntity categoryEntity = categoryDao.selectById(catelogId);
            if(categoryEntity!=null){
                attrResVo.setCatelogName(categoryEntity.getName());
            }

            //获得attr——group——name需要一张中间表和attr——group的表，所以导入两个Dao
            AttrAttrgroupRelationEntity entity = attrAttrgroupRelationDao.selectOne(new QueryWrapper<AttrAttrgroupRelationEntity>().eq("attr_id", obj.getAttrId()));
            if(entity!=null){
                AttrGroupEntity attrGroupEntity = attrGroupDao.selectById(entity.getAttrGroupId());
                attrResVo.setGroupName(attrGroupEntity.getAttrGroupName());
            }
            return attrResVo;
        }).collect(Collectors.toList());


        PageUtils pageUtils = new PageUtils(page);
        pageUtils.setList(res);
        return pageUtils;

```
* 3.修改APIs首先需要正确回显，逆向生成的回显实体类是```AttrEntity```，少了属性分组和属性分类，同样，修改成使用```AttrResVo```来回显。属性分类是一个级联选择器，需要后端返回一个完整的路径数组。最后确定修改的按钮使用update的controller函数。但是如果一开始在新增时候没有选择属性分组。这里按update就不能成功，因为一开始需要是一个新增操作。所以用selectCount来判断。当一个查询语句查出来的count>0时，说明这张表里面原来就已经有了这个值，这样就执行update，否则执行insert。
```
@Transactional
@Override
public void updateAttr(AttrVo attr) {

    AttrEntity attrEntity = new AttrEntity();
    
    //copy the value to attrEntity, and update
    BeanUtils.copyProperties(attr,attrEntity);
    this.updateById(attrEntity);

    //update pms_attr_attrgroup_relation table
    Long attrGroupId = attr.getAttrGroupId();
    AttrAttrgroupRelationEntity attrAttrgroupRelationEntity = new AttrAttrgroupRelationEntity();
    attrAttrgroupRelationEntity.setAttrId(attr.getAttrId());
    attrAttrgroupRelationEntity.setAttrGroupId(attrGroupId);


    Integer count = attrAttrgroupRelationDao.selectCount(new UpdateWrapper<AttrAttrgroupRelationEntity>().eq("attr_id", attr.getAttrId()));
    if(count>0){
        //修改操作
        attrAttrgroupRelationDao.update(
                attrAttrgroupRelationEntity,
                new UpdateWrapper<AttrAttrgroupRelationEntity>().eq("attr_id",attr.getAttrId())
        );
    }else{
       //新增操作 
       attrAttrgroupRelationDao.insert(attrAttrgroupRelationEntity);

    }

}

```
* 4.完成销售属性和规格参数的转换，销售属性的显示就是和规格参数用的同一张```psm_attr```表，```attrType```中1表示规格参数，0表示销售属性。发现销售属性的请求url与规格参数不同，所以在后端时使用```@RequestMapping("/{attrType}/list/{categoryId}")```来区分。然后再上述的修改查询新增里面加入判断是否为1即可。
* 5.编写一个constant类来储存```attrType```的1和0，这样对后续的修改很方便，而不需要再具体类中找寻。
```
public class ProductConstant {
    public enum AttrEnum{
        ATTR_TYPE_BASE(1,"基本属性"),ATTR_TYPE_SALE(0,"销售属性");
        private int code;
        private String msg;
        AttrEnum(int code,String msg){
            this.code=code;
            this.msg=msg;
        }

        public int getCode() {
            return code;
        }

        public String getMsg() {
            return msg;
        }
    }

}

```
### 2021.02.21
* Finish the SPU and SKU CRUD like product release including across different services by using FeignClient method.
### 2021.02.22-23
* Finish the Ware system design including show the purchase products,purchase detail products and ware CRUD.
* Finish the update operation attributes of product  based on SPU Id in Product System.

# Finish the Base Model of Gulimall


