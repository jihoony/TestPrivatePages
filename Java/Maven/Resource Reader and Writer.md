

# Resource Reader and Writer

## Resource Reader 샘플 코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.Reader;
import java.util.Objects;

public class ResourceReaderFile {

    private BufferedReader bufferedReader;

    public ResourceReaderFile(String fileName) {
        bufferedReader = new BufferedReader(new InputStreamReader(Objects.requireNonNull(this.getClass().getResourceAsStream(fileName))));
    }

    public Reader getReader() {
        return bufferedReader;
    }

    public String readLine() {
        try {
            return bufferedReader.readLine();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```



## Resource Writer 샘플 코드

```java
import java.io.*;

public class ResourceWriterFileImpl {

    private BufferedWriter bufferedWriter;

    public ResourceWriterFileImpl(String fileName){
        String filePath = fileName;

        File file = new File(filePath);
        if (!file.exists()){
            try {
                file.createNewFile();
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }

        try {
            bufferedWriter = new BufferedWriter(new FileWriter(file));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    public void writeLine(String line){
        try {
            bufferedWriter.write(line);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    public void close(){
        try {
            bufferedWriter.flush();
            bufferedWriter.close();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

