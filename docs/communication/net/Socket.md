# Socket

Socket 是一组操作 TCP/UDP 的 API，像 HttpURLConnection 和 Okhttp 这种涉及到比较底层的网络请求发送的，最终当然也都是通过 Socket 来进行网络请求连接发送，而像 Volley、Retrofit 则是更上层的封装。

## 使用示例

1. 创建 ServerSocket 并监听客户连接。
2. 使用 Socket 连接服务端。
3. 通过 Socket.getInputStream()/getOutputStream() 获取输入输出流进行通信。

```java
public class EchoClient {
    private final Socket mSocket;

    public EchoClient(String host, int port) throws IOException {
        // 创建 socket 并连接服务器
        mSocket = new Socket(host, port);
    }

    public void run() {
        // 和服务端进行通信
        Thread readerThread = new Thread(this::readResponse);
        readerThread.start();

        OutputStream out = mSocket.getOutputStream();
        byte[] buffer = new byte[1024];
        int n;
        while ((n = System.in.read(buffer)) > 0) {
            out.write(buffer, 0, n);
        }
    }

    private void readResponse() {
        try {
            InputStream in = mSocket.getInputStream();
            byte[] buffer = new byte[1024];
            int n;
            while ((n = in.read(buffer)) > 0) {
                System.out.write(buffer, 0, n);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] argv) {
        try {
            // 由于服务端运行在同一主机，这里我们使用 localhost
            EchoClient client = new EchoClient("localhost", 9877);
            client.run();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
