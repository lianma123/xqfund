## springboot 2.0 集成 mybatis

### 环境：

* 开发工具：Intellij IDEA 2017.1.3
* springboot: **2.0.1.RELEASE**
* jdk：1.8.0_40
* maven:3.3.9
* alibaba Druid 数据库连接池：1.1.9

### 额外功能：

* PageHelper 分页插件
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>兴全基金统计</title>
    <script src="https://cdn.bootcss.com/jquery/3.1.1/jquery.min.js"></script>
    <script src="js/echarts.min.js"></script>
</head>
<body>
<!-- 为ECharts准备一个具备大小（宽高）的Dom -->
<div id="main" style="width: 600px;height:400px;"></div>
<script type="text/javascript">
    // 基于准备好的dom，初始化echarts实例
    var myChart = echarts.init(document.getElementById('main'));

    // 指定图表的配置项和数据
    myChart.setOption({
        title: {
            text: '兴全基金吧点击回复柱状图'
        },
        tooltip: {},
        legend: {
            data:['销量']
        },
        xAxis: {
            data: []
        },
        yAxis: {},
        series: [{
            name: '销量',
            type: 'bar',
            data: []
        }]
    });

    myChart.showLoading();

    var authors=[];    //类别数组（实际用来盛放X轴坐标值）
    var clicks=[];    //销量数组（实际用来盛放Y坐标值）
    var replys=[];

    $.ajax({
        type : "get",
        async : false,            //异步请求（同步请求将会锁住浏览器，用户其他操作必须等待请求完成才可以执行）
        url : "user/list",    //请求发送到TestServlet处
        data : {},
        dataType : "json",        //返回数据形式为json
        success : function(result) {
            //请求成功时执行该函数内容，result即为服务器返回的json对象
            var data = result.data;
            if (data) {
                for(var i=0;i<data.length;i++){
                    authors.push(data[i].author);    //挨个取出类别并填入类别数组
                }
                for(var i=0;i<data.length;i++){
                    clicks.push(data[i].click);    //挨个取出销量并填入销量数组
                }
                for(var i=0;i<data.length;i++){
                    replys.push(data[i].reply);    //挨个取出销量并填入销量数组
                }
                myChart.hideLoading();    //隐藏加载动画
                myChart.setOption({        //加载数据图表
                    xAxis : [
                        {
                            type : 'category',
                            data : authors
                        }
                    ],
                    yAxis : [
                        {
                            type : 'value'
                        }
                    ],
                    series : [
                        {
                            name:'点击量',
                            type:'bar',
                            data: clicks
                        },
                        {
                            name:'回复量',
                            type:'bar',
                            data: replys
                        }



                    ]
             /*       xAxis: {
                        data: authors
                    },
                    series: [{
                        // 根据名字对应到相应的系列
                        name: '点击量',
                        data: clicks
                    },
                        {
                            name: '回复量',
                            data: replys
                        }]*/
                });

            }

        },
        error : function() {
            //请求失败时执行该函数
            alert("图表请求数据失败!");
            myChart.hideLoading();
        }
    })
</script>
</body>
</html>



mapper
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.winterchen.dao.UserDao" >
  <sql id="BASE_TABLE">
    xq_user
  </sql>

  <sql id="BASE_COLUMN">
    userId,click,reply,title,author,senddata,lastdata
  </sql>

  <insert id="insert" parameterType="com.winterchen.model.UserDomain">
    INSERT INTO `xq_user`(click,reply,title,author,senddata,lastdata) values (#{click},#{reply},#{title},#{author},#{senddata},#{lastdata})
<!--      <include refid="BASE_TABLE"/>
    <trim prefix="(" suffix=")" suffixOverrides=",">
      click,reply,title,
      <if test="click != null">
        author,senddata,lastdata,
      </if>
    </trim>
    <trim prefix="VALUES(" suffix=")" suffixOverrides=",">
      #{click, jdbcType=INTEGER},#{reply, jdbcType=INTEGER}, #{title, jdbcType=VARCHAR},
      <if test="click != null">
        #{author, jdbcType=VARCHAR}, #{senddata, jdbcType=VARCHAR}, #{lastdata, jdbcType=VARCHAR},
      </if>
    </trim>-->
  </insert>

<!--  <select id="selectUsers" resultType="com.winterchen.model.UserDomain">
      SELECT
        <include refid="BASE_COLUMN"/>
      FROM
        <include refid="BASE_TABLE"/>
  </select>-->
  <select id="selectUsers" resultType="com.winterchen.model.UserDomain">
  select author,click,reply from `xq_user` limit 10
  </select>

</mapper>


controller

package com.winterchen.controller;

import com.github.pagehelper.PageHelper;
import com.winterchen.model.UserDomain;
import com.winterchen.service.user.UserService;
import com.winterchen.service.user.impl.UserServiceImpl;
//import com.winterchen.utils.Scrap;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import com.winterchen.model.UserDomain;
import com.winterchen.service.user.UserService;
import com.winterchen.service.user.impl.UserServiceImpl;
import org.jsoup.Jsoup;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;
/**
 * Created by Administrator on 2017/8/16.
 */
@Controller
@RequestMapping(value = "/user")
public class UserController {

    @Autowired
    private UserService userService;
    @Autowired
    private UserServiceImpl userServiceImpl;
    @RequestMapping(value = "", method = RequestMethod.GET)
    public String index() {
        return "index";
    }
    @RequestMapping(value = "list", method = RequestMethod.GET)
    @ResponseBody
    public Map<String,Object> getList() {
        Map<String,Object> map = new HashMap<>();
        map.put("msg", "ok");
        map.put("data", userService.findAll());
        return map;
    }

/*    @RequestMapping(value = "", method = RequestMethod.GET)
    public String index() {
        return "index";
    }*/
    @ResponseBody
    @GetMapping("/add")
    public void addUser(){
            for(int i=1;i<429;i++){
            String url = "http://guba.eastmoney.com/type,zg80036742_"+i+".html";
            Document doc = null;
            try {
                UserDomain userDomain = new UserDomain();
                doc = Jsoup.connect(url).get();
                Elements elements = doc.select(".pager").select("span");
//                System.out.println(elements);
                Elements listDiv = doc.getElementsByAttributeValue("class", "articleh");
//            System.out.println(listDiv);

                for(Element element : listDiv){

                    Elements texts = element.getElementsByTag("a");
                    Elements texts1 = element.select(".l6");
                    Elements texts2 = element.select(".l5");
                    Elements texts3 = element.select(".l4");
                    Elements texts4 = element.select(".l1");
                    Elements texts5 = element.select(".l2");
//                System.out.println(texts1);
                    for(Element text:texts4){
                        String ptext = text.text();
                        if(ptext!="") {
//                            System.out.print("点击："+ptext+"	");
                            userDomain.setClick(Integer.valueOf(ptext));
                        }

                    }
                    for(Element text:texts5){
                        String ptext = text.text();
                        if(ptext!="") {
//                            System.out.print("点击："+ptext+"	");
                            userDomain.setReply(Integer.valueOf(ptext));
                        }

                    }
                    for(Element text:texts3){
                        String ptext = text.text();
                        if(ptext!="") {
                            userDomain.setAuthor(ptext);
                        }

                    }
                    for(Element text:texts2){
                        String ptext = text.text();
                        if(ptext!="") {
                            userDomain.setLastdata(ptext);
                        }

                    }
                    for(Element text:texts1){
                        String ptext = text.text();
                        if(ptext!="") {
                            userDomain.setSenddata(ptext);
                        }

                    }
                    for(Element text:texts){
                        String ptext = text.attr("title");
                        if(ptext!="") {
                            userDomain.setTitle(ptext);

                        }





                    }
                    userService.addUser(userDomain);
                }
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }

        }}


    @ResponseBody
    @GetMapping("/all")
    public Object findAllUser(
            @RequestParam(name = "pageNum", required = false, defaultValue = "1")
                    int pageNum,
            @RequestParam(name = "pageSize", required = false, defaultValue = "10")
                    int pageSize){
        return userService.findAllUser(pageNum,pageSize);
    }
}
