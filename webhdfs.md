```python
# CRATE A FILE

url = "http://localhost:9870/webhdfs/v1/user/student/ncd_screen.csv?op=CREATE&user.name=student"
r = requests.put(url, data=open('ncd_screen_001.csv', 'r').read())
r.status_code
```
