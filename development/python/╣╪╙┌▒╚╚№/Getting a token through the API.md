[Official link: 2024.1 API references](https://docs.openstack.org/2024.1/api/)

```python
import requests
import json

# 获取 Token 的 URL
auth_url = 'http://10.5.167.102:5000/v3/auth/tokens'

# 认证信息
auth_body = {
    "auth": {
        "identity": {
            "methods": ["password"],
            "password": {
                "user": {
                    "name": "admin",
                    "domain": {"name": "demo"},
                    "password": "000000"
                }
            }
        }
    }
}

auth_header = {"Content-Type": "application/json"}

def get_token():
    try:
        response = requests.post(auth_url, json=auth_body, headers=auth_header)
        if response.status_code == 201:
            token = response.headers['X-Subject-Token']
            print(f"Token 获取成功: {token}")
            return {"X-Subject-Token": token, "Content-Type": "application/json"}
        else:
            print(f"Token 获取失败, 状态码: {response.status_code}")
            return None
    except Exception as e:
        print(f"请求失败: {e}")
        exit(1)

class Image:
    def __init__(self, url, headers):
        self.url = url
        self.headers = headers
        self.body = {
            "container_format": "bare",
            "disk_format": "qcow2",
            "name": "cirros001"
        }

    def new_image(self):
        try:
            # 先创建镜像记录
            response = requests.post(self.url, headers=self.headers, json=self.body)
            if response.status_code == 201:
                image_id = response.json().get('id', '未知ID')
                print(f"镜像创建成功，ID为: {image_id}")
                
                # 上传镜像文件
                upload_url = f"{self.url}/{image_id}/file"
                with open("/root/cirros-0.3.4-x86_64-disk.img", "rb") as image_data:
                    upload_response = requests.put(upload_url, headers={"X-Subject-Token": self.headers["X-Subject-Token"]}, data=image_data)
                    if upload_response.status_code == 204:
                        print("镜像上传成功")
                    else:
                        print(f"镜像上传失败, 状态码: {upload_response.status_code}")
            else:
                print(f"镜像创建失败，状态码: {response.status_code}")
        except Exception as e:
            print(f"创建镜像时出错: {e}")

if __name__ == '__main__':
    token_headers = get_token()
    if token_headers:
        image = Image(url='http://10.5.167.102:9292/v2/images', headers=token_headers)
        image.new_image()

```