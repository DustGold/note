

 # Default 生命周期 

### Default 生命周期是 Maven 生命周期中最重要的一个，绝大部分工作都发生在这个生命周期中。这里， 只解释一些比较重要和常用的阶段： 

###### validate 

###### generate-sources

###### process-sources 

###### generate-resources 

##### process-resources 复制并处理资源文件，至目标目录，准备打包。 

##### compile 编译项目的源代码。 

###### process-classes 

###### generate-test-sources

###### process-test-sources 

###### generate-test-resources

##### process-test-resources 复制并处理资源文件，至目标测试目录。

##### test-compile 编译测试源代码。 process-test-classes

##### test 使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。 

###### prepare-package

##### package 接受编译好的代码，打包成可发布的格式，如 JAR。

##### pre-integration-test 

###### integration-test 

###### post-integration-test 

###### verify 

##### install 将包安装至本地仓库，以让其它项目依赖。 

##### deploy 将最终的包复制到远程的仓库，以让其它开发人员与项目共享或部署到服务器上运行。 