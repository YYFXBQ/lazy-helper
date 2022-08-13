# 懒人小帮手

想要变懒

## 天气获取

#### 城市获取

作为一个懒人脚本，首先肯定是要自动获取相关数据，手动输入那也太麻烦了。

```
import json
import requests


def get_location():
    url = "http://httpbin.org/ip"  # 访问这个地址
    r = requests.get(url)  # 获取返回的值
    ip = json.loads(r.text)["origin"]  # 取其中某个字段的值

    # 发送get请求
    url = f'http://ip-api.com/json/{ip}?fields=status,message,country,countryCode,region,regionName,city,zip,lat,lon,' \
          f'timezone,isp,org,as,query&lang=zh-CN'
    # 其中fields字段为定义接受返回参数，可不传；lang为设置语言，zh-CN为中文，可以传
    res = requests.get(url)  # 发送请求
    data = json.loads(res.text)
    print(data)
    return data['city']
```

这里通过两个请求

一：爬取，当前设备的ip地址（此处爬到的ip并不是局域网内的ip）；二：通过这个ip地址获取当前的物理地址，可以显示国家城市等；~~其实只要城市就够了~~



**8.11**修改

由于之前使用的接口好像被墙了，使用国内的ip不太方便登上去所以换了一个接口

```python
def get_location():
    """获取当前ip的地址省份"""
    try:
        payload = {}
        headers = {}

        response = requests.request("GET", url, headers=headers, data=payload)
        myjson = json.loads(response.text)
        # print(myjson['province'])
        # print(myjson['city'])
        return myjson['city'][:2]
    except requests.exceptions.ConnectionError:
        get_location()
        # requests.exceptions.ConnectionError
```



### 天气获取

'http://wthrcdn.etouch.cn/weather_mini?city=%s' 通过这个api接口获取json数据。

```
def get_weather():
    city_name = get_location()  # 默认获取当前城市
    city_code = cities[city_name]  # 这里我爬了国内的城市名称与相对应的城市id

    # url = 'http://wthrcdn.etouch.cn/weather_mini?city=%s' % city_name  # 查询可以直接用城市名
    url = 'http://wthrcdn.etouch.cn/weather_mini?citykey=%s' % city_code
    # 也可以使用城市代码，话说我可以用城市名为啥用城市代码.做都做了

    res = requests.get(url)
    info = res.json()
    data = info['data']  # 数据
    forecast = data['forecast']  # 天气预报的相关数据
    current_temperature = data['wendu']  # 当前的温度
    today_forcast = forecast[0]
    print("城市： %s" % city_name + "\n天气：%s" % today_forcast['type'] + "\n当前温度：%s" % current_temperature
          + f"\n{today_forcast['low']}  {today_forcast['high']}" + "\n风向: %s" % today_forcast['fengxiang'] +
          "\t 风力： %s" % today_forcast['fengli'][9:11] + "\ntips: " + data['ganmao'])
```

json数据格式如下：

```
# {'yesterday':
#      {'date': '26日星期日', 'high': '高温 35℃', 'fx': '南风', 'low': '低温 27℃', 'fl': '<![CDATA[2级]]>', 'type': '阴'},
#  'city': '杭州',
#  'forecast':
#      [{'date': '27日星期一', 'high': '高温 36℃', 'fengli': '<![CDATA[3级]]>', 'low': '低温 28℃', 'fengxiang': '南风', 'type': '阴'},
#       {'date': '28日星期二', 'high': '高温 37℃', 'fengli': '<![CDATA[2级]]>', 'low': '低温 28℃', 'fengxiang': '南风', 'type': '阴'},
#       {'date': '29日星期三', 'high': '高温 38℃', 'fengli': '<![CDATA[2级]]>', 'low': '低温 28℃', 'fengxiang': '南风', 'type': '小雨'},
#       {'date': '30日星期四', 'high': '高温 32℃', 'fengli': '<![CDATA[2级]]>', 'low': '低温 25℃', 'fengxiang': '南风', 'type': '小雨'},
#       {'date': '1日星期五', 'high': '高温 33℃', 'fengli': '<![CDATA[3级]]>', 'low': '低温 24℃', 'fengxiang': '东风', 'type': '小雨'}],
#  'ganmao': '感冒低发期，天气舒适，请注意多吃蔬菜水果，多喝水哦。', 'wendu': '35'}
```

好，首先，这样对于天气数据请求获取的部分，应该差不多了吧。后面咱总不能老是打开py程序看吧，弄个前端界面，打包个exe还是有必要的。



## tkinker做个能看的前端

### 简单的加了几个label和按钮

```python
class MY_GUI:
    # 初始化
    def __init__(self, init_window_name):
        super().__init__()
        self.init_window_name = init_window_name
        self.init_window_name.title("懒人小帮手")  # 窗口名
        self.init_window_name.geometry('600x350')  # 窗口大小及初始化位置

        # 文本框
        self.city_name_label = Label(self.init_window_name, text="输入城市名")
        self.city_name_label.grid(row=0, column=0)
        self.forcast_show_label = Label(self.init_window_name, text="天气预测结果")
        self.forcast_show_label.grid(row=0, column=3)
        self.city_name = Text(self.init_window_name, width=10, height=2)  # 城市名录入框
        self.city_name.grid(row=1, column=0, rowspan=1, columnspan=1)  # tkinker都是网格时布局，长宽多少格，以及部署在第几格
        self.forcast_show = Text(self.init_window_name, width=60, height=20)  # 处理结果展示
        self.forcast_show.grid(row=1, column=3, rowspan=10, columnspan=5)
        # 按钮

        self.str_trans_to_md5_button = Button(self.init_window_name, text="获取天气情况", bg="lightblue", width=10,
                                              command=self.get_weather)  # 调用内部方法  加()为直接调用
        self.str_trans_to_md5_button.grid(row=1, column=2)

    # 设置窗口
    def set_init_window(self):
        local_city = get_location()
        self.city_name.insert(1.5, local_city)  # 把获取到的当前城市名插入到文本框内
        self.get_weather()
        self.the_timer() # 启动定时器
        self.init_window_name.mainloop()

    # 功能函数 获取天气
    def get_weather(self):
        city_name = self.city_name.get(1.0, END).strip()
        # city_name = get_location()  # 默认获取当前城市
        city_code = cities[city_name]  # 这里我爬了国内的城市名称与相对应的城市id

        # url = 'http://wthrcdn.etouch.cn/weather_mini?city=%s' % city_name  # 查询可以直接用城市名
        url = 'http://wthrcdn.etouch.cn/weather_mini?citykey=%s' % city_code
        # 也可以使用城市代码，话说我可以用城市名为啥用城市代码.做都做了

        res = requests.get(url)
        info = res.json()
        data = info['data']  # 数据
        # forecast = data['forecast']  # 天气预报的相关数据
        # current_temperature = data['wendu']  # 当前的温度
        # today_forcast = forecast[0]
        show_txt = ("城市：%s" % city_name + "\n天气：%s" % data['forecast'][0]['type'] + "\n当前温度：%s" % data['wendu']
              + "度" + f"\n{data['forecast'][0]['low']}  {data['forecast'][0]['high']}" + "\n风向: %s" % data['forecast'][0]['fengxiang'] +
              "\t 风力： %s" % data['forecast'][0]['fengli'][9:11] + "\ntips: " + data['ganmao'])

        self.forcast_show.delete(1.0, END)  # 把之前显示的内容删除
        self.forcast_show.insert(1.0, show_txt)

    # 定时器
    def the_timer(self):
        now = time.strftime("%H:%M:%S")
        print(now)
        if now[3:5] == "30":
            self.get_weather()
        else:
            pass
        self.init_window_name.after(60000, self.the_timer)  # 草有（）就立即执行了


def gui_start():
    init_window = Tk()  # 实例化出一个父窗口

    ZMJ_PORTAL = MY_GUI(init_window)

    ZMJ_PORTAL.set_init_window()  # 初始化窗口


if __name__ == '__main__':
    gui_start()
```

init中初始化了些文本框和按钮，tkinker都是以网格布局分布的，由于个人审美不大行，做的前端是真的丑。

然后把get_weather 变成内部函数，真坑啊（内部函数只要加上（）就会立即执行，怪不得我定时器一写就在无线触发）

将获取到的数据显示在show_txt 中显示出来

写了个定时器，每小时30分时触发一次获取天气。



发现有个问题是：请求天气时，api返回较慢导致界面打开较慢而且有可能会报error。下面来处理这几个问题。



### 解决问题

1.先打开界面再去请求天气

~~不想加其他的变量，加其他的变量，程序打开是快了，但是只有下一次定时器触发的时候才会返回当前的天气~~

该怎么弄呢，没啥想法，后面接着再优化吧



2.进行了下异常处理：

首先查表看是否有对应的城市id存在，

如果没有就在使用城市名查询，

如果城市名返回的不符合格式就输出城市名错误。

```python
try:
    city_code = cities[city_name]  # 这里我爬了国内的城市名称与相对应的城市id
    url = 'http://wthrcdn.etouch.cn/weather_mini?citykey=%s' % city_code
except KeyError:
    url = 'http://wthrcdn.etouch.cn/weather_mini?city=%s' % city_name
# 也可以使用城市代码，话说我可以用城市名为啥用城市代码.做都做了
try:
    res = requests.get(url)
    info = res.json()
    data = info['data']  # 数据
    # print(data)
    # forecast = data['forecast']  # 天气预报的相关数据
    # current_temperature = data['wendu']  # 当前的温度
    # today_forcast = forecast[0]
except KeyError:
    show_txt = "城市名输入错误，请重新输入"
    self.forcast_show.insert(1.0, show_txt)
else:
    show_txt = ("城市：%s" % city_name + "\n今天天气：%s" % data['forecast'][0]['type'] + "\n当前温度：%s" % data['wendu']
          + "度" + f"\n{data['forecast'][0]['low']}  {data['forecast'][0]['high']}" + "\n风向: %s" %
          data['forecast'][0]['fengxiang'] + "\t 风力： %s" % data['forecast'][0]['fengli'][9:11] + "\ntips: "
          + data['ganmao'])
    show_txt1 = ("\n未来天气：\n"
                 + data['forecast'][1]['date'] + "  天气：" + data['forecast'][1]['type'] + " " + data['forecast'][1]['low']
                 + data['forecast'][1]['high'] + '\n'
                 + data['forecast'][2]['date'] + "  天气：" + data['forecast'][2]['type'] + " " +data['forecast'][2]['low']
                 + data['forecast'][2]['high'])
```





## 新增需求

1.好bro说，背景随着天气变化就好了。还想要动态的图片

2.我想把界面缩小到托盘。



### 需求处理

1.学习一下Canvas组件

Could not find a version that satisfies the requirement PIL (from versions: none)

PIL库不能安装的原因：

现在已经用Pillow代替PIL，PIL较多用于2.7版本的Python中



解决方案：

pip install Pillow

**tkinter只支持gif格式，在展示jpg或png等格式，需使用PIL库获得ImageTk.PhotoImage对象代替tk.PhotoImage对象。**



tkinker 窗口弄成透明好弄，但是控件透明化好像并不好弄。

使用动图的话，需要以它的帧数实时刷新图片。



2.tk缩小到托盘

参考： https://blog.csdn.net/wodeyan001/article/details/82497564#comments_20843287

有段时间没更新了都有点忘了

目前效果如下：图片可以是动图也可以是普通图片

<img src="懒人小助手.assets/image-20220801095029270-16593186364441.png" alt="image-20220801095029270" style="zoom: 80%;" />

可以有缩放功能。

![image-20220801095141196](懒人小助手.assets/image-20220801095141196.png)

## 更换图标

https://www.qtool.net/ico

普通图片转ico图片的网址

```python
def __init__(self, icon, hover_text, menu_options, on_quit, tk_window=None, default_menu_index=None,
             window_class_name=None):
    '''
    icon         需要显示的图标文件路径
    hover_text   鼠标停留在图标上方时显示的文字
    menu_options 右键菜单，格式: (('a', None, callback), ('b', None, (('b1', None, callback),)))
    on_quit      传递退出函数，在执行退出时一并运行
    tk_window    传递Tk窗口，s.root，用于单击图标显示窗口
    default_menu_index 不显示的右键菜单序号
    window_class_name  窗口类名
    '''
    self.icon = '2020.ico'
    self.hover_text = hover_text
    self.on_quit = on_quit
    self.root = tk_window
```

init 中设置初始的属性







## pyinstaller 打包

在虚拟环境中，安装好需要的包。

```
(venv) PS E:\Python project\lazy helper> pyinstaller -F -w -i picture/懒人助手.ico main.py -p cities.py -p picture -n 懒人小助手
```

进行打包。















本文参考到了：

https://blog.csdn.net/xff123456_/article/details/124174936

