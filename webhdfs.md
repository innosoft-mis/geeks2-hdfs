```python
# CRATE A FILE

url = "http://localhost:9870/webhdfs/v1/user/student/ncd_screen.csv?op=CREATE&user.name=student&overwrite=true"
r = requests.put(url, data=open('ncd_screen_001.csv', 'r').read())
r.status_code

# FILE STATUS

url = "http://localhost:9870/webhdfs/v1/user/student/ncd_screen.csv?op=GETFILESTATUS&user.name=student"
r = requests.get(url)
obj = json.loads(r.content)
print(json.dumps(obj, indent=2))

# APPEND FILE

url = "http://localhost:9870/webhdfs/v1/user/student/ncd_screen.csv?op=APPEND&user.name=student"
r = requests.post(url, data=open("ncd_screen_002.csv","r").read())
r.status_code

```
