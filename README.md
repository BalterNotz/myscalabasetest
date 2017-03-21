# myscalabasetest

mvn clean scala:compile compile package

如上，在compile前加入scala:compile，这是maven-scala-plugin插件提供的选项，表示编译scala，这样一来，先编译scala，再编译java，最后打包.