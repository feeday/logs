通过修改配置文件的代码可以改文章字数。

- 文件位置代码：wp-includes/formatting.php 3323 行

- 切换到配置文件查找代码位置

```
cd /wp-includes/formatting.php
grep -rn "excerpt_length = apply_filters( 'excerpt_length', 55 )" *
```

- 查找替换命令 55 改成 56。
- 关键字 length’, 55  替换  length’, 56

```
sed -i s/"length', 55"/"length', 56"/g `grep "length', 55" -rl --include="formatting.php" ./`
```