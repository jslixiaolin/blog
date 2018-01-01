# 去除字符串重复部分

昨天笔试有这么一个问题，**去除一个字符数组的重复部分，不能开辟额外的空间。**
思路大概如下：
利用字符串结束符 ‘\0’,和一个额外的下标index，从而实现原来字符数组的复用。
* 对数组遍历，如果不为'\0'，做第二步
* 将当前元素的值赋给下标为index的数组，index往后移动一位
* 对当前元素之后的所有元素进行遍历，如果碰到和当前元素相同的，将其置为 '\0'
* 对数组遍历结束，index对应的数组值置 '\0'
* 这时候就得到了去重的字符数组，结束位置为index

代码如下：

```
public static void remove3(char[] array) {
    if (array == null || array.length <= 0) {
      return;
    }
    int len = array.length;
    int index = 0;
    for (int i = 0; i < len; i++) {
      if (array[i] != '\0') {
        array[index++] = array[i];

        for (int j = i + 1; j < len; j++) {
          if (array[j] == array[i]) {
            array[j] = '\0';
          }
        }
      }
    }
    array[index] = '\0';
    int k = 0;
    StringBuffer sb = new StringBuffer();
    while (array[k] != '\0') {
      sb.append(array[k++]);
    }
    System.out.println(sb.toString());

  }
```

测试用例：
1、连续重复的：aabbccdd
2、非连续重复：abbjkjsab
