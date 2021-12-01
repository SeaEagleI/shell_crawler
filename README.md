# R语言大作业：基于贝壳找房网的北京房源爬虫与房源房价分析

## 项目目标

- 目标城市：北京（城市编号110000，城市缩写bj）
- 以二手房房源分析为主，租房、新房房源分析为辅

## Part I: 数据获取（API爬虫）

### 数据源网址

- 贝壳找房：https://www.ke.com
- 北京二手房：https://bj.ke.com/ershoufang/
- 北京租房：https://bj.zu.ke.com/zufang/
- 北京新房：https://bj.fang.ke.com/loupan/

### 爬虫结构

- 列表爬虫：通过贝壳api获取房源编号及基本信息列表
- 详情页爬虫：通过房源编号得出贝壳详情页url，进入详情页爬取户型、建筑结构、小区、首付等更多信息
- 地理坐标爬虫：通过高德地图api根据每条房源的小区名获得房源的近似地理经纬度（在预处理代码中实现）

### 爬虫代码及爬取结果展示

|                             功能                             | 爬取耗时 | 爬取房源数 |                   网站实时房源数                   | API数据重复性 |     网站数据重复性     | 爬取结果去重 |
| :----------------------------------------------------------: | :------: | :--------: | :------------------------------------------------: | :-----------: | :--------------------: | :----------: |
| [获取二手房房源](https://github.com/SeaEagleI/house_price_analysis/blob/master/ershoufang.py) | 约40分钟 |   88653    | [88653](https://bj.ke.com/ershoufang/)（11月19日） |    无重复     |         无重复         |     ---      |
| [获取租房房源](https://github.com/SeaEagleI/house_price_analysis/blob/master/zufang.py) | 约40分钟 |   36069    |  [38069](https://bj.zu.ke.com/zufang/)（11月7日）  |    有重复     |          ---           |    已去重    |
| [获取新房房源](https://github.com/SeaEagleI/house_price_analysis/blob/master/newhouse.py) |  约10秒  |    261     |  [261](https://bj.fang.ke.com/loupan/)（11月7日）  |    有重复     | 有重复（存在一房多挂） |    未去重    |

### 说明
- 运行相应python程序即可在data目录下生成格式为json的房源信息文件，文件中房源信息数如上表所示。
- 以上房源数仅为程序实时获得的结果，而贝壳网的房源数据是持续变化的，重新运行程序即可得到包含最新房源数量和信息的文件。
- 租房API提供数据存在较多冗余，使用"house_code"字段对房源去重后为36069条，与网站实时结果差2k左右。原因具体是以下哪种仍有待研究，但不影响后期对数据的使用和分析：
    1. API提供的数据不完整；
    2. 贝壳网租房信息本身存在冗余（如一房多挂）。
- 项目全部数据见[wps共享文件夹](https://kdocs.cn/join/gi5qoxj)，链接永久有效。（涉及数据量较大，git限制上传的单个文件大小上限是50MB）

## Part II: 数据预处理

- [文件格式转换](https://github.com/SeaEagleI/house_price_analysis/blob/master/preprocess/json2csv.py)：
    1. 将爬取的json文件转为csv（递归处理树结构数据、使用路径重命名字段名）
- [数据整理](https://github.com/SeaEagleI/house_price_analysis/blob/master/preprocess/ershoufang.py)：
    1. 删除冗余列、无用列；
    2. 修改列值：删除列值中的单位，并拆分、合并一些列；
    3. 重命名列名：将英文换成中文（增加可读性），在列名中加上单位。

## Part III: 房源房价统计及对比分析

- 房源特征统计、区域比较分析（整体上/每个区域哪些特征的房子最多: 楼层？价格？房屋类型 ==> 柱状图、词云图）
    1. csv文本词频统计 => **词云图**
    2. 房源基本信息统计
        - 房屋用途分布 => **饼图**
        - 房屋户型分布 => **饼图**
        - 装修情况分布 => **饼图**
        - 配备电梯分布 => **饼图**
        - 近地铁分布 => **饼图**
        - 区域分布 => **饼图**
        - 所在楼层分布 => **饼图**
        - 楼层总数分布 => **饼图**
        - 建筑面积分布 => **饼图**
    3. 房源价格信息统计及与其他因素的比较分析
        - 北京各区域二手房房源数量、平均单价、平均建筑面积比较关系 => **并列柱状图**
        - 北京二手房单价与建筑面积关系 => **散点图**
        - 北京二手房单价Top10小区 => **并列柱状图**
        - 北京各区域二手房单价数值分布 => **箱线图**
        - 北京各区域二手房总价数值分布 => **箱线图**

- 房源、房价的区位分布：
    1. 房源区位分布 => **区域热力图**
    2. 房价区位分布 => **区域热力图**

- 二手房、租房、新房间的横向对比：
    1. 三者间总数对比 （二手房交易频繁的区域租房房源也是最多吗？） => **并列柱状图**
    2. 同商圈的新房与二手房单价、面积对比 => **并列折线图**
    - 总结：想买房主要还是买二手房（新房房源太少且**比二手房单价更高、面积更大、更买不起**，地产商无利不起早；租房=打工，从长远来看不合算）
    3. ~~有没有房子既是二手房也是出租房？（二手房和租房的houseCode不统一，而通过其他字段很难判断一条二手房房源和一条租房房源是否是同一房源，所以该问题没有可行性，又不属于甲方实际需求，故删掉）~~

## Part IV: 房源聚类

- 房源聚类（聚类指标：房屋面积、单价、地理位置），对每类房源给出基本特征、地理特征描述，并分析：
    1. 从地图上看每类房源的位置分布，分析房源类型分布与区位的关系，如学区房小房子多、郊区房大房子多； => **区域热力图**
    2. 对单价影响最大的因素有哪些，如区域、商圈、学区、楼层、房屋面积、建房时间等。

## Part V: Case Study

- 购房案例研究：
    1. 参数估计：在现实条件下，首付与总价差多少能保证还得起？（假设：30年还清差价，家庭年还30w，公积金共140w，30年还清）
    2. 使用上步得到的结果，在各个区安家分别需要多少首付？（目标房价取本区的房价中位数）
    3. 每年还款数额分别为10w、20w、30w时，最低首付如何变化？

- 租房案例研究：
    1. 本部燕园校区附近的高性价比租房房源？(假设通勤距离、租金、居住条件为主要考虑因素，在租房数据中选择出三者均合适的性价比房源)
    2. 给出推荐清单，并在ppt中展示top2

## 参考

- **贝壳、链家网爬虫**
  - https://www.zhihu.com/question/443457100/answer/1721778654
  - https://github.com/CaoZ/Fast-LianJia-Crawler
  - https://zhuanlan.zhihu.com/p/370244126

- **数据分析与可视化**
  - https://github.com/ideaOzy/data_analysis
  - https://blog.51cto.com/u_15168725/2708108
  - https://zhuanlan.zhihu.com/p/359589517

- **Python代码转R**
  - https://github.com/Mounment/R-Project/tree/master/上海二手房分析

## 声明
本项目仅供学习交流使用，严禁使用本项目作商业用途，违者后果自负。
