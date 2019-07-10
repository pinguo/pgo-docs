# 单元测试(UnitTesting)

支持goPath模式和go module模式,运行成功后会在项目目录下生成coverage文件夹，文件夹下生成coverage.html,coverage.txt两个报表文件

## 配置
```shell
1.拷贝文件github.com/pinguo/pgo/gotest.sh 到自己项目根目录，也可以不用拷贝,执行的时候需要给不同路径

2.打开gotestl.sh文件,修改arr_test_path_dir_name变量

// 修改需要测试的包 goPath环境下直接填写包名 go module环境下填写 module_name/package_name
arr_test_path_dir_name=(
    Command
    Controller
    Lib
    Model
    Service
    Struct
    Test
)
```


## 运行

```shell
1. bash gotest.sh  启动文件gotest.sh在项目根目录文件下  
2. bash github.com/pinguo/pgo/gotest.sh /home/worker    自定义项目目录，目录下有conf配置文件夹
3. bash github.com/pinguo/pgo/gotest.sh /home/worker go test ./     自定义项目目录，目录下有conf配置文件夹 ,自定义命令运行(不需要修改arr_test_path_dir_name变量)
```

## 单元测试代码编写

```go
1.如果不涉及GetObject使用，请按常规方法编写
2.涉及GetObject使用:
    // Lib包
    func TestDdd_GetName(t *testing.T) {
        // UnitTesting.TestObj全局变量
        ddd:= UnitTesting.TestObj.GetObject("Lib/Ddd").(*Ddd)
        ret := ddd.GetName()
        if ret!="ddd"{
           t.FailNow()
        }
        t.Log("ok:")
    }

    // Service包
     func TestAaa_a1(t *testing.T)  {
         // UnitTesting.GetTestObj()每次初始化一个对象 
         aaa:= UnitTesting.GetTestObj().GetObject("Service/Aaa").(*Aaa)
         ret := aaa.a1()
         if ret!="aaa"{
             t.FailNow()
         }
         t.Log("ok:")
     } 
```



