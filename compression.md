# GZIP decompression

```java
import java.util.Base64;
import java.nio.charset.StandardCharsets;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.util.zip.GZIPInputStream;
import java.io.IOException;



public class MyClass {
    static String decompress(byte[] arrayBytes) throws IOException {
        final int bufferSize = 32;
        ByteArrayInputStream inputStream = new ByteArrayInputStream(arrayBytes);
        GZIPInputStream gis = new GZIPInputStream(inputStream, bufferSize);
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        byte[] data = new byte[bufferSize];
        int bytesRead;
        while ((bytesRead = gis.read(data)) != -1) {
            baos.write(data, 0, bytesRead);
        }
        gis.close();
        inputStream.close();
        return baos.toString(StandardCharsets.UTF_8);
    }
  
    public static void main(String args[]) {
      int x=10;
      int y=25;
      int z=x+y;

        try {
            String data = "H4sIAAAAAAAA/2WQz2rcMBDGX8XobBfZlu1d30KyoYY2KdSG0lKEVh6nIvIfJHmT7ZJHaEtuvZQl0EIvOZWEkBz9Jn2B7iNU7oYupSehb775zTezQrU+QekKnbXqtJLtGYVz4L0RbUNFiVKfxAQTFwkDihmgTV/PQaEUW0lTA9qgtGJSg4t2jdqM3pMlStHs1Wy/yGfIRSUzbBykWFO2tSVjjCcuMnBuEWiz/vbZGe6Hh8368saph3stOHM264+3DpfD1ePv8gNoW7x2KrYYfihhthZLN8JIsJxsNOnhaj58l87zsciUGa4beO/8/PL1192nfx/b2WtQ21XDKE4SEsYXLtIGOqu4qOvnUuh31KYf8QEOQg9PPUxyf5L6URpETyY+eW05nRKtTTQufTA73Cue5VZctJIZIYHyttluurJ0xv+cqSiyA+vuek2rqEF/9d0hHx1lEmM/CcHjQex7BJfcY9V07k0rUiVAQu4T2LWbZTdGfVG8fEqPjvPsMNvfy7PjI+uoW35K/5uiQPfSaJS+eWvDlSDFAtSSGlFbjh9Pw0kUBEF08Rs/wjJELAIAAA==";
            String data1 = "H4sIAAAAAAAA/2WQz2rcMBDGX8XobBfZsrVr30KyoYY2KdSG0lKE7JVTEfkPkrzJdskjtCW3XsoSaKGXnEpCSI5+k75A9xE67oYupSehb775zTezQrU5QckKnbX6tFLtGRPnouytbBsm5yjxQxpi6iJpheZWsKavC6FRgkEyzApjUVJxZYSLdo3Gjt6TJUrQ7NVsP89myEVzbvk4SPNm3tZAxtgHihXngECb9bfPznA/PGzWlzdOPdwbWXJns/5465RquHr8XX4QBorXTsUXww8t7dYCdCutEsBJR5MZrorhu3Kej0Wu7XDdiPfOzy9ff919+veBzt4IvV2VRHQyCQm9cJGxogPFRV1fKGneMUg/4gMcEA/HHg4zf5r4UULwE+rT18DptGwh0bj0wexwL3+WgbhoFbdSCVa2zXbTFdB5+edMeZ4egLvrDauiBv3Vd4d8dASExDiKqBcXVeiFpCi8KY4nnk9xVYW8LGlAdu122Y1RX+Qvn7Kj4yw9TPf3svT4CBx1W56y/6ZoYXplDUrevIVwc6HkQugls7IGjk9jMo2CgOCL31sXbUgsAgAA";
            System.out.println(decompress(Base64.getDecoder().decode(data1)));
        } catch(Exception ex) {
          System.out.println(ex);  
        }
    }
}
```
