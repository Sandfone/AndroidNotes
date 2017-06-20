# Android gradle配置（持续更新中……）

-------------------------

- Android 打包自动生成“version code”

  ```groovy
  android {
      // 根据git commit number自动生成version code
      def cmd = 'git rev-list HEAD --first-parent --count'
      // 解决新版本build号低于老版本应用build number而导致的无法覆盖更新的情况
      def gitVersion = cmd.execute().text.trim().toInteger() + 100
      defaultConfig {
          versionCode gitVersion
  //        versionCode cdvVersionCode ?: new BigInteger("" + privateHelpers.extractIntFromManifest("versionCode"))
          applicationId privateHelpers.extractStringFromManifest("package")

          if (cdvMinSdkVersion != null) {
              minSdkVersion cdvMinSdkVersion
          }
      }
  }
  ```

- Android 打包自定义安装包名称

  ```groovy
  android {
      // 配置打包apk名称
      applicationVariants.all { variant ->
          def application = "mcrm"
          def buildName = "" //渠道名字
          def outApkName = "mcrm.apk" //最终输出文件名
          def outDir = "./"

          def versionName = 'v' + variant.getVersionName() //版本名称
          def versionCode = 'c' + variant.getVersionCode()//版本号

          variant.outputs.each { output ->
              variant.productFlavors.each { product ->
                  buildName = product.name //获取渠道名字
              }
              outDir = output.outputFile.parent
              outApkName = application + '_' + versionName + '_' + versionCode + '_' + variant.buildType.name + buildName + '.apk'
              output.outputFile = new File(outDir, outApkName)
          }
      }
  }
  ```

  ​

  ​