# Shell之if_elif_else

## 一、语法
```shell
#!/bin/bash

if [ $1 == "start" ]; then
  echo "start..."
  exit 0
elif [ $1 == "stop" ]; then
  echo "stop..."
  exit 0
else
  echo "nothing..."
  exit 0
fi
```

## 注意点1：[ ] 左右两侧需有空格

## 注意点2：若then与if、elif同行，then之前需加分号;

## 注意点3：定义变量时, =号的两边不可以留空格

## 注意点4：字符串比较, =两边要留空格

## 参考
[1] https://blog.csdn.net/ai2000ai/article/details/82353591