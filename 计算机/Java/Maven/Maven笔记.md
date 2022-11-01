# Maven

## Maven内部文件标签

### \<packaging\>标签

\<package\>表示打包类型，一般父工程的打包类型为pom，子工程打包类型为jar，默认为jar。



### \<DependencyManagement\>标签

\<DependencyManagement\>标签主要用在父类工程中管理依赖，这样子类再次引入相同依赖时，不需要再次声明版本号，如果整个项目需要修改依赖版本，只需要在父类中进行更换；而子类需要另一个版本号，则可以在子类中声明版本号。