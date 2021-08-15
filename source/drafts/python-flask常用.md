### 开启虚拟环境
```
py -3 -m venv vevn  #创建虚拟环境
.\venv\Scripts\Activate.ps1 #激活虚拟环境
```

### 安装需要的包

```
pip install packagename #单个包安装
pip install -r requirements.txt #将所有需要的包放在文件中，从文件中读取安装
```


### pytest测试
```python
pytest          #执行所有测试
pytest xxx.py   #执行指定文件内所有测试
```