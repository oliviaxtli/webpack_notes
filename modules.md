## Modules
在模块化编程中，开发者将程序分解为功能离散的chunk，并称之为模块。每个模块都拥有小于完整程序的体积，使得验证、调试以及测试变得轻而易举。

## 模块解析
resolver是一个帮助寻找模块绝对路径的库。一个模块可以作为另一个模块的依赖模块，然后被后者引用
```
import foo from 'path/to/module';
// 或者
require('path/to/module');
```
所依赖的模块可以是来自应用程序的代码或第三方库。resolver帮助webpack从每个require/import语句中，找到需要引入到bundle中的模块代码。当打包模块时，webpack使用`enhanced-resolve`来解析文件路径

### webpack中的解析规则
使用`wenhanced-resolve`，webpack能解析三种文件路径
#### 绝对路径
```
import '/home/me/file';

import 'C:\\Users\\me\\file';
```
由于已经获得文件的绝对路径，因此不需要再做进一步解析

#### 相对路径
```
import '../src/file1';
import './file2';
```
使用import或require的资源文件所处的目录，被认为是上下文目录。在import/require中给定的相对路径，会拼接此上下文路径，来生成模块的绝对路径

#### 模块路径
```
import 'module';
import 'module/lib/file';
```
在resolve.modules中指定的所有目录中检索模块。可以通过配置别名的方式来替换初始模块路径`resolve.alias`。


一旦根据上述规则解析路径后，resolver会检查路径是指向文件还是文件夹。如果路径指向文件：
- 如果文件具有扩展名，则直接将文件打包
- 否则，将使用`resolve.extensions`选项作为文件扩展名来解析，此选项会告诉解析器在解析中能够接受哪些扩展名

如果路径指向一个文件夹，则进行如下步骤寻找具有正确扩展名的文件：
- 如果文件夹中包含package.json文件，则会根据resolve.mainFields配置中的字段顺序查找，并根据package.json中的符合配置要求的第一个字段来确定文件路径
- 如果不存在package.json文件或resolve.mainFields没有返回有效路径，则会根据resolve.mainFiless配置选项中指定的文件名顺序查找，看能否在import/require的目录下匹配到一个存在的文件名
- 然后使用resolve.extensions选项，以类似的方式解析文件扩展名
