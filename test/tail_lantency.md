---
sort: 4
---

# 尾部时延

***这是一个用python实现并发尾时延探测的一个记录***

原理很简单，利用多线程并发请求，只需要修改并发数量和IP地址即可，但是不知道为什么并发数多了效果很差。

```
#客户端，自己指定IP
import socket
import time
import concurrent.futures

def measure_latency(local_ip, host, port):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
        sock.bind((local_ip, 0))  # 绑定到本地IP和任意端口
        sock.connect((host, port))
        start_time = time.time()
        sock.sendall(b'Hello')
        response = sock.recv(1024)
        end_time = time.time()
        return 1000*(end_time - start_time)

def perform_test(local_ip, host, port, num_requests):
    with concurrent.futures.ThreadPoolExecutor(max_workers=num_requests) as executor:
        latencies = list(executor.map(lambda _: measure_latency(local_ip, host, port), range(num_requests)))
        return latencies

# 测试参数
LOCAL_IP = '192.168.0.41'  # 替换为实际的本地IP地址
HOST = '192.168.0.42'            # 替换为服务器的IP地址
PORT = 12345
NUM_REQUESTS = 200

latencies = perform_test(LOCAL_IP, HOST, PORT, NUM_REQUESTS)
latencies.sort()
for latency in latencies:
    print(latency)
print("Average latency:", sum(latencies) / len(latencies))
print("95th percentile latency:", sorted(latencies)[int(0.95 * len(latencies))])

```

```
#服务器端
import socket
import threading

def handle_client_connection(client_socket):
    while True:
        message = client_socket.recv(1024)
        if not message:
            break
        # 响应客户端
        client_socket.sendall(b'ACK')

def start_server(host='0.0.0.0', port=12345):
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind((host, port))
    server_socket.listen(5)
    print(f'Server listening on {host}:{port}')

    while True:
        client_sock, address = server_socket.accept()
        print(f'Accepted connection from {address[0]}:{address[1]}')
        client_handler = threading.Thread(target=handle_client_connection, args=(client_sock,))
        client_handler.start()

if __name__ == '__main__':
    start_server()


```
