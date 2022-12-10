# Maven

`maven`安装jar包到本地`maven`仓库

```bash
mvn install:install-file \
   -Dfile=./libs/utils.jar \
   -DgroupId=cn.microvideo.lab \
   -DartifactId=utils \
   -Dversion=1.0 \
   -Dpackaging=jar \
   -DgeneratePom=true
```
