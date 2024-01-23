---
title: "linux tensorflow-java"
categories:
  - "development"
tags:
  - "java"
  - "tensorflow"
date: "2023-01-01T01:01:00+09:00"
comments: true
toc: true
sidebar: false
draft: true
---


```bash
apt update
apt -y upgrade
apt install -y openjdk-8-jdk
apt install -y wget unzip vim
```

```bash
wget https://services.gradle.org/distributions/gradle-7.5.1-bin.zip -P /tmp
unzip -d /opt/gradle /tmp/gradle-*.zip
```

```bash
/etc/profile.d/gradle.sh
```

```sh 
export GRADLE_HOME=/opt/gradle/gradle-7.5.1
export PATH=${GRADLE_HOME}/bin:${PATH}
```

```bash
chmod +x /etc/profile.d/gradle.sh
source /etc/profile.d/gradle.sh
```


```java
import org.tensorflow.ConcreteFunction;
import org.tensorflow.Signature;
import org.tensorflow.Tensor;
import org.tensorflow.TensorFlow;
import org.tensorflow.op.Ops;
import org.tensorflow.op.core.Placeholder;
import org.tensorflow.op.math.Add;
import org.tensorflow.types.TInt32;

public class App {
    public String getGreeting() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        System.out.println(new App().getGreeting());
        System.out.println("Hello TensorFlow " + TensorFlow.version());

        try (ConcreteFunction dbl = ConcreteFunction.create(App::dbl);
            TInt32 x = TInt32.scalarOf(10);
            Tensor dblX = dbl.call(x)) {
            System.out.println(x.getInt() + " doubled is " + ((TInt32)dblX).getInt());
        }
    }

    private static Signature dbl(Ops tf) {
        Placeholder<TInt32> x = tf.placeholder(TInt32.class);
        Add<TInt32> dblX = tf.math.add(x, x);
        return Signature.builder().input("x", x).output("dbl", dblX).build();
    }

}
```

```gradle
implementation group: 'org.tensorflow', name: 'tensorflow-core-platform', version: '0.3.3'
```

