# 使用MAT分析Android内存泄露
## 步骤
### 1
Android hprof文件生成可以使用adb shell am dump < pid >< output-file-name>  ,再通过android sdk中的hprof-conv工具进行转换
命令:hprof dump.hprof converted-dump.hprof
使用MAT打开hropf文件
![open](1.png)
### 2
点击查看"OQL"根据OQL语句查询,在编辑框内输入"select * from instanceof android.app.Activity",在点击红色感叹号按钮执行
![oql](2.png)
### 3
得到应用中所有未被回收的Activity列表
![leak](3.png)
### 4
任意选择一个你关系的类,右键点击"Merge Shortest Paths to GC Root",在弹出二级菜单中选择"excudle all phantom/weak/soft etc. references"
![merge](4.png)
### 5
得到泄露引用列表
![result](5.png)
