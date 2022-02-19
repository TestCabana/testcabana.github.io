---
title: 学习了一下接口框架DDT数据的传递过程
categories: Python
tags:
  - Python
---

#### 学习了一下接口框架DDT数据的传递过程

##### 话说在前面

- 在之前的接口脚本中，用csv文件、xlsx文件作为载体存放接口用例的数据。现在学习一下xmind，个人觉得xmind的结构：一个主题下面有多个子主题，在一个接口多种场景（正常值、空值、错误值）下，可以让用例文件更直观。
- 但我自己也只是大概的debug了一下，原来的代码没啥备注，啃得可能也有些错误，有大佬指点一下的话感激涕零~



##### 一、xmind结构

![image-20211110142019391](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20211110142019391.png)

xmind文件名（cyTest.xmind）

画布：等同于excel中sheet的概念，默认名为：画布 1

主题：一个画布由一个中心主题开始延伸，中心主题下包含子主题。

主题的一些要素

1. title标题：分支主题2
2. note备注：备注2

3. markers标记：priority-2优先级、tag-orange标签颜色、smiley-cry情绪、task-start任务完成度、等

4. labels标签：标签二

5. link链接：http://www.baidu.com
6. topics子主题：子主题也是相同的要素



##### 二、python的xmindparser模块常用方法

```python
import xmindparser
# xmindparser配置
xmindparser.config = {
            'showTopicId': False, #是否展示主题ID
            'hideEmptyValue': True， #是否隐藏空值
            'showStructure': False, #是否展示结构值
            'showRelationship': False #是否展示节点关系
        }
filePath = r'/文件路径/cyTest.xmind'
data_json = xmindparser.xmind_to_json(filePath) #解析成json数据类型
data_dcit = xmindparser.xmind_to_dict(filePath) #解析成dict数据类型 - 常用
data_xml = xmindparser.xmind_to_xml(filePath) #解析成xml数据类型 
# 解析到文件
file_json = xmindparser.xmind_to_file(filePath, 'json')
file_xml = xmindparser.xmind_to_file(filePath, 'xml')
#[{'title': '画布 1', 'topic': {'title': '中心主题', 'topics': [{'title': '分支主题 1', 'note': '备注一', 'makers': ['priority-1', 'tag-red', 'smiley-laugh', 'task-done'], 'labels': ['标签一','标签二'], 'topics': [{'title': '子主题 1', 'makers': ['flag-red']}, {'title': '子主题 2'}]}, {'title': '分支主题 2', 'note': '备注二', 'makers': ['priority-2', 'tag-orange', 'smiley-cry', 'task-start'], 'labels': ['标签二'], 'link': 'http://www.baidu.com', 'topics': [{'title': '子主题 3'}]}]}, 'structure': 'org.xmind.ui.map.unbalanced'}]
```



##### 三、公司的xmind用例结构

<img src="C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20211111194307394.png" alt="image-20211111194307394" style="zoom:200%;" />

一级主题，作为class名，所以尽量用首字母大写的英文单次（必须英文字母或下划线开头、字母/数字/下划线组成，如果有其他字符会被替换为"_"）

```python
def to_safe_name(string):
    return str(re.sub("[^a-zA-Z0-9_]+", "_", string)
```



##### 四、框架中读取xmind接口测试数据的代码解析

这一部门的代码我全部copy到下面了，因为自己之前ddt没有用过xmind来作为数据载体，所以手写了一遍去理解xmind模板为什么要这么写，觉得太长的话就直接看最后函数把接口数据整理成什么样的格式就行。

```python
from xmindparser import xmind_to_dict
import json

HOST_TAG_CHOICE = ["mk", "qw", "bill", "qyapi"]  # 对应 host_<mk/qw>
PRIORITY_TAG_CHOICE = ["critical", "blocker", "normal"]

class ReadXmind(object):
    """读取xmind内容，返回字典列表（list[dict1,dict2]）"""
    def __init__(self, file_path, canvas=""):
        self.file_path = file_path
        self.canvas = canvas
        self.xmind_dict_list = xmind_to_dict(file_path)

    def get_canvas_name_list(self):
        '''遍历xmind文件的每一个画布，得到全部画布的名称装成列表 ['画布 1', '画布 2']，但是如果传入了canvas，就不重新读直接return'''
        return [self.canvas] if self.canvas else [cv.get('title') for cv in self.xmind_dict_list]

    def get_canvas_topic(self, canvas="画布 1"):
        """
        根据初始指定的画布名称（对应于excel的sheet表），返回该画布的接口数据
        :return: {'title': '画布中心主题', 'topics': [{接口1信息},{接口2信息},...]} 此时title为画布 1 的中心主题 BillTrailSettingApi
        """
        for cv in self.xmind_dict_list: #遍历每一个画布
            if cv.get('title') == canvas: #如果画布名==传入的画布名参数
                return cv.get('topic') #return画布下的全部接口[{接口1信息},{接口2信息},...]
        else:
            raise Exception("Xmind中画布<{}>不存在!".format(canvas))

    def get_canvas_data(self, canvas):
        """
        获取画布内所有API-CASE信息
        :return: canvas_data
        case_dict 字典：存放一个用例的信息，eg用例名event_001、用例描述desc等
        api_dict 字典：存放一个接口的信息，eg接口名api_name、依赖depends、所有用例test_case_list [{case_dict},{case_dict},...]
        api_case_dict_list 数组：存放一个画布的所有接口信息，[{api_dict},{api_dict},...]
        canvas_data 字典：存放一个画布的信息，{'desc': '画布 1', 'module': 'BillTrailSettingApi', 'api_case_list':[{api_dict},{api_dict},...]} 
        """
        canvas_data = {} #dict 存放：画布名、中心主题、api_case_list接口列表
        api_case_dict_list = [] #list 存放：api_dict
        canvas_topic = self.get_canvas_topic(canvas)
        module_name = canvas_topic.get('title')
        canvas_data['desc'] = canvas #画布名
        canvas_data['module'] = module_name #中心主题 BillTrailSettingApi
        api_list = canvas_topic.get('topics') or [] #接口列表 [{接口1信息},{接口2信息},...]
        for api in api_list: #遍历每一个接口
            api_dict = {} #存放：接口信息
            api_name = api.get('title') if api.get('title') else "" #api名 event
            api_desc = api.get('note') if api.get('note') else "" #描述 新增事件
            api_labels = api.get('labels')  #标签 depends、host、sleep
            api_dict['api_name'] = api_name
            api_dict['api_desc'] = api_desc
            api_dict['depends'] = []  # depends 依赖标签
            api_dict['sleep'] = 0  # sleep 标签
            api_dict['skipif'] = ""  # skipif 标签
            host_tag = "host_" + HOST_TAG_CHOICE[0]  # host 标签 默认为qw
            # 如果有接口数据有标签，重新赋值，否则就是上述空值和默认值
            if api_labels:
                for alb in api_labels: #遍历每个标签 'labels': ['host=qw', 'depends=addUser', 'sleep=2', 'skipif=skip', 'name=name']
                    lb_dps = alb.split("depends=") #返回['host=qw'] 子字符串在父字符串中不存在时，返回整个父字符串作为列表的元素
                    lb_dps2 = alb.split("name=")
                    lb_sleep = alb.split("sleep=")
                    lb_skipif = alb.split("skipif=")
                    lb_host = alb.split("host=") #返回['','qw']
                    if len(lb_dps) == 2:
                        api_dict['depends'].append(lb_dps[-1])
                    if len(lb_dps2) == 2:
                        api_dict['depends'].append(lb_dps2[-1])
                    if len(lb_sleep) == 2:
                        api_dict['sleep'] = int(lb_sleep[-1])
                    if len(lb_skipif) == 2:
                        api_dict['skipif'] = lb_skipif[-1]
                    if len(lb_host) == 2 and lb_host[-1] in HOST_TAG_CHOICE:
                        host_tag = "host_" + lb_host[-1]
            api_dict['host_tag'] = host_tag
            api_dict['test_case_list'] = []  #存放：该接口的全部用例
            api_detail = api.get('topics')[0]
            path = api_detail.get('title') or "" #接口路径
            rq = api_detail.get('topics')[0]
            method = rq.get('title') or "" #请求方法
            api_dict['path'] = path
            api_dict['method'] = method
            case_cf_list = rq.get('topics') #一个接口的多个条用例 [{用例1},{用例2},...]
            if api_name.startswith('#'):  #被注释掉的接口，不解析用例参数
                continue
            for idx, case in enumerate(case_cf_list): #enumerate将可遍历的数据对象组合为一个索引序列
                if case.get('title').startswith('#'):  #被注释掉的用例，不解析
                    continue
                case_dict = {} #存放：用例信息
                case_labels = case.get('labels')  # 用例描述下的标签
                case_dict['module'] = module_name  # 模块名
                case_dict['api_name'] = api_name  # api名
                case_dict['host_tag'] = host_tag  # host_qw / host_mk
                case_dict['api_desc'] = api_desc  # api描述
                case_dict['depends'] = api_dict['depends']  # api依赖
                case_dict['sleep'] = api_dict['sleep']  # api sleep
                case_dict['skipif'] = api_dict['skipif']  # api skipif
                case_dict['name'] = f"{api_name}_" + str(idx+1).zfill(3)  # case名 zfill填充字符串为三位 event_001 event_002
                case_dict['desc'] = case.get('title')  # 用例描述
                case_dict['priority'] = ""  # 用例优先级
                if case_labels: #标签为 用例严重程度
                    for clb in case_labels:
                        if clb in PRIORITY_TAG_CHOICE:
                            case_dict['priority'] = clb
                            break
                case_dict['method'] = method  # method: get/post/...
                case_dict['path'] = path  # api path
                case_input_output = case.get('topics')[0] #入参及后面的子主题
                case_dict['data'] = case_input_output.get('title')  #入参
                output_set_vars = case_input_output.get('topics')
                str_expect = output_set_vars[0].get('title') #期望结果
                try:
                    case_dict['expect'] = json.loads(str_expect, strict=False)  #xmind读出来的期望结果str转换成字典
                # 自定义异常 定位到主题名-api名-期望结果
                except Exception as e:
                    print("API:{0}->{1}\nJSON Content:{2}".format(module_name, api_name, str_expect))
                    raise e
                case_dict['set_vars'] = {}
                if len(output_set_vars) > 1: #期望结果有两个主题 即设置了全局的变量
                    str_set_vars = output_set_vars[1].get('title') #拿到变量'{"customerId": "$..data.id"}'
                    try:
                        case_dict['set_vars'] = json.loads(str_set_vars, strict=False)  # 需要设置为全局的变量，key-value， key为保存变量名，value为字段查询名或jsonpath
                    #自定义异常 变量写法有问题 定位到主题名-api名-变量
                    except Exception as e:
                        print("API:{0}->{1}\nJSON Content:{2}".format(module_name, api_name, str_set_vars))
                        raise e
                api_dict['test_case_list'].append(dict(case_dict))
            api_case_dict_list.append(api_dict)
        canvas_data['api_case_list'] = api_case_dict_list
        return canvas_data

    def get_xmind_data(self):
        '''读xmind每一个画布的数据 [{canvas_data},{canvas_data},...]'''
        return [self.get_canvas_data(cv) for cv in self.get_canvas_name_list()]
'''
case_dict的值，之后会作为pytest的传参
{'module': 'BillTrailSettingApi', 'api_name': 'event', 'host_tag': 'host_qw', 'api_desc': '新增事件', 
'depends': ['addUser', 'name'], 'sleep': 2, 'skipif': 'skip', 
'name': 'event_001', 'desc': '异常值（新增事件参数为异常值校验）', 'priority': 'critical', 'method': 'POST', 
'path': '/bill-trail-settings-api/trail/event', 'data': '{"eventIcon": "事件图标", "eventName": "事件名称", "eventTopic": "事件", "fieldList": "事件内容列表", "id": "事件ID，编辑时必传", "operator": "操作人", "remark": "备注", "trailId": "旅程项目ID"}', 
'expect': {'code': 8401, 'msg': '参数不能为空', 'data': None}, 'set_vars': {'customerId': '$..data.id'}}
'''
```

一个点：因为之前没找到完整的xmind模板，不明白为什么api_name主题下的标签，depends=  和 name=  要存在一个数组depends中，最后悟了。方便dependency去传参，这样没有依赖，却需要别名给其他接口依赖时，depends传入空数组[]。妙！（dict的value存数组也完全ok）

```python
dp_name, depends = None, []
for dp in api_data['depends']: #["addEevent","event"] api_name=event
    if dp == api_name: #如果apiname是当前自己接口的名字，作为别名
        dp_name = api_name
        continue
	else:
		depends.append(dp)
if dp_name or len(depends) > 0:
	logger.info("dependency: {} : name={},depends={}".format(func_name, dp_name, depends))
	attrs[func_name] = pytest.mark.dependency(name=dp_name, depends=depends, scope='class')(attrs[func_name])
# skip以#开头的测试用例
if api_name.startswith("#") or api_desc.startswith("#"):
	attrs[func_name] = pytest.mark.skip(reason='测试用例被注释#')(attrs[func_name])
```



##### 五、接口脚本自动生成的大概原理和过程

所用技术：文件写入脚本、compile()内置函数编译、metaclass对子类的改写、装饰器与闭包函数，和一些杂七杂八的

上面写到了DDT的测试数据xmind载体的解析，拿到了参数之后传入pytest即可（题外话pytest两种传参方式，fixture或者parametrize，一定要知晓参数传递过程的），接下来写一下作者大大将参数融进pytest的过程，可能自己理解的比较简陋，有大佬指点的话感激涕零~

第一步：构建用例脚本，就是with open去写就ok了，主要写入的是：导包的内容、类模板（包括脚本名称和测试数据），compile后生成脚本的类如下所示~

```python
@allure.feature('画布 1')
class TestBillTrailSettingApi(PreRequest, metaclass=CaseMetaClass):
    test_py = "test_bill-trail-settings-api.py"
    test_cases_data = [{接口1},{接口2},...]
```

PreRequest这个父类就是一些请求前的操作，比如拿变量值、sessionId，就不展开了。

第二步：compile生成函数对象，用例的模板字符串参数传入api_name，来动态生成接口函数。模板有很多执行时的细节操作，比如skip、allure敏捷标记等，不详细展开了，贴一下生成的函数代码。如下~

```python
def create_function(function_express, namespace=None):
    """动态生成函数对象"""
    if namespace is not None:
        builtins.__dict__.update(namespace) # 哇 全部添加到内建模块中
    module_code = compile(function_express, '', 'exec')  # 根据模板(字符串)生成可执行的代码，mode为exec（源包含多个python语句时使用）
    function_code = [x for x in module_code.co_consts if isinstance(x, types.CodeType)][0] #注意isinstance()会考虑继承关系 所以这里用type()不行哟！
    return types.FunctionType(function_code, builtins.__dict__) #返回内存地址<function test_001_event at 0x05A0AA98>
```

第三步：重点来了！！写一个metaclass，来修改子类的属性或方法

```python
class CaseMetaClass(type):
    def __new__(cls, name, bases, attrs):
        # print("此时new的对象是",cls.__name__) #CaseMetaClass
        test_cases_data = attrs.pop('test_cases_data') # 字典的pop有返回值 拿到一个画布的接口数据[{接口1},{接口2},...]
        for idx, api_data in enumerate(test_cases_data):
            if is_contains_chinese(api_data['api_name']) and not is_contains_chinese(api_data['api_desc']): #检查了一手api_name是否包含中文，decs为英文，是就互换一下
                api_data['api_name'], api_data['api_desc'] = api_data['api_desc'], api_data['api_name']
            api_name = api_data['api_name']
            api_name = "" if api_name is None else api_name #又让api_name兼容一手None值
            api_desc = api_data['api_desc']
            case_list = api_data['test_case_list'] #一个接口的用例列表 [{case_dict},{case_dict},...]
            func_name = 'test_{0}_{1}'.format(str(idx+1).zfill(3), to_safe_name(api_name.replace("#", ""))) #api_name=event格式统一成 test_001_event
            function = create_function(func_template.format(func_name=func_name),
                                       namespace={
                                           'pytest': pytest,
                                           'allure': allure,
                                           'logger': logger,
                                           'is_contains_chinese': is_contains_chinese,
                                           'sleep_progressbar': sleep_progressbar,
                                           'methodcaller': methodcaller,
                                           'setup': setup,
                                           'callback': callback,
                                       })
            ids = [str(c).zfill(3) for c in range(1, len(case_list)+1)] # 接口有多少条用例 001 002 003...
            # 哇 这个写法又学习了，返回的function内存地址后，直接手工闭包！和用parametrize装饰器一样哟，nice！
            attrs[func_name] = pytest.mark.parametrize('case_data', case_list, ids=ids)(function)
            dp_name, depends = None, []
            for dp in api_data['depends']:
                if dp == api_name:
                    dp_name = api_name
                    continue
                else:
                    depends.append(dp)
            if dp_name or len(depends) > 0:
                logger.info("dependency: {} : name={},depends={}".format(func_name, dp_name, depends))
                attrs[func_name] = pytest.mark.dependency(name=dp_name, depends=depends, scope='class')(attrs[func_name]) #装饰一手 依赖
            if api_name.startswith("#") or api_desc.startswith("#"):
                attrs[func_name] = pytest.mark.skip(reason='测试用例被注释#')(attrs[func_name]) #再装饰一手skip 基本齐活，坐等调用func_name，pytest就嗖嗖运行起来惹，妙！
            # 集成allure装饰器 这个不知道是不是作者大大觉得模块下面展示case就行了，再打story好懒得挨个点开喔，我是这么想的~
            # attrs[func_name] = allure.story(api_data['api_desc'])(attrs[func_name])
        return super().__new__(cls, name, bases, attrs)
```

一些点：

1、pytest在收集用例时，按照test_ 开头去找，自动生成的脚本就是test_ 文件名.py，测试用例集按照类来展示一个xmind文件每个画布的接口用例，比如上面的class TestBillTrailSettingApi()，pytest执行时按顺序去解析类，会先调用CaseMetaClass.__ new __()来创建类，所以CaseMetaClass可以借助attrs来改变TestBillTrailSettingApi()类的属性或方法。

2、metaclass我的理解是the class of the class，类的类，它new出来的是一个类，而非实例。自定义的CaseMetaClass通过new方法返回子类的name、bases、attrs即类名、继承的集合、属性方法。这是meclass与继承不一样层面的原因，子类继承父类除私有属性和方法外的特性，对父类进行重写，但父类无法操纵子类，借用metaclass，就可以对子类进行操作，像装饰器一样去【动态修改】。（关于metaclass这篇还可以：https://www.cnblogs.com/yssjun/p/9832526.html?ivk_sa=1024320u，多多debug试一试理解更快）

3、在上述代码的new方法中，通过“手工闭包”来形成装饰器，将函数内存地址传给pytest.mark.parametrize()，等同于平常使用@pytest.mark.parametrize()。下面写了一个最最最简单的闭包函数，当然还有装饰方法、装饰器传参等各种情况就不细说了。只是简单举个栗子~

```python
 def outer(func):
        def inner():
            print("before")
            func()
            print("after")
        return inner
    
    @outer
    def cytest():
        print("陈语测试")
    cytest() #装饰后调用原函数
    
    #这等价于上面的装饰
    cytest = outer(cytest)
    cytest() #调用闭包函数inner
```

4、当xmind的一个画布由多个接口，new方法在for循环中遍历每个接口，根据接口api_name来使子类动态生成多个function，即动态生成了多个接口用例集了，如下图。

![image-20211120151106113](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20211120151106113.png)

##### 六、写不动了

以上就是大致把数据流看了一下，做的笔记。

这样run_main主入口脚本的运行过程就可以串起来啦
