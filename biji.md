# 笔记

## 1. 启动项目

执行初始化

- windows：参考MAKEFILE文件中的目标进行执行
```
go get -u google.golang.org/protobuf/cmd/protoc-gen-go
go get -u google.golang.org/grpc/cmd/protoc-gen-go-grpc
go get -u github.com/go-kratos/kratos/cmd/protoc-gen-go-http/v2
go get -u github.com/google/wire/cmd/wire
go get -u github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2
go install github.com/envoyproxy/protoc-gen-validate@latest


.PHONY: grpc
# generate grpc code
grpc:
	 cd api/admin/v1 && protoc --proto_path=. \
           --proto_path=../../../third_party \
           --proto_path=../../../pkg/proto \
           --go_out=paths=source_relative:. \
           --go-grpc_out=paths=source_relative:. \
           $(API_PROTO_FILES)

.PHONY: http
# generate http code
http:
	cd api/admin/v1 && protoc --proto_path=. \
           --proto_path=../../../third_party \
           --proto_path=../../../pkg/proto \
           --go_out=paths=source_relative:. \
           --go-http_out=paths=source_relative:. \
           --validate_out=paths=source_relative,lang=go:. \
           --openapi_out=fq_schema_naming=true,default_response=false:../../../. \
           $(API_PROTO_FILES)

.PHONY: errors
# generate errors code
errors:
	cd api/admin/v1 && protoc --proto_path=. \
           --proto_path=../../../third_party \
           --proto_path=../../../pkg/proto \
           --go_out=paths=source_relative:. \
           --go-errors_out=paths=source_relative:. \
           $(API_PROTO_FILES)

.PHONY: config
# generate internal proto
config:
	cd app/admin && protoc --proto_path=./internal \
		--proto_path=../../third_party \
		--proto_path=../../pkg/proto \
		--go_out=paths=source_relative:./internal \
		$(INTERNAL_PROTO_FILES)

.PHONY: swagger
# generate swagger
swagger:
	cd api/admin/v1 && protoc --proto_path=. \
	       --proto_path=../../../third_party \
	       --proto_path=../../../pkg/proto \
	       --openapiv2_out . \
	       --openapiv2_opt logtostderr=true \
	       --openapiv2_opt allow_merge=true \
	       --openapiv2_opt merge_file_name=api.proto \
	       --openapiv2_opt json_names_for_fields=false \
	       --openapiv2_opt=openapi_configuration=swagger.config.yaml \
           $(API_PROTO_FILES)

.PHONY: proto
# generate internal proto struct
proto:
	cd app/admin && protoc --proto_path=. \
           --proto_path=../../third_party \
           --proto_path=../../pkg/proto \
           --go_out=paths=source_relative:. \
           $(INTERNAL_PROTO_FILES)
```
- linux
```
make init
make all
```

前置配置：
```
-//     protoc-gen-go v1.30.0           官方给定的版本
-//     protoc        v4.23.2           官方给定的版本
+//     protoc-gen-go v1.27.0           本地安装的版本
+//     protoc        v3.17.3           本地安装的版本
```
需要安装PROTOC和PROTOC-GEN-GO



配置文件：app/admin/configs/config.yaml
1. 配置数据库
2. 配置REDIS

配置文件路径
```
flag.StringVar(&flagconf, "conf", "D:\\workspace\\golang\\src\\github.com\\byteflowteam\\kratos-vue-admin\\app\\admin\\configs\\config.yaml", "config path, eg: -conf config.yaml")
```

关闭GOOGLE验证码：
```
-       gAuth := util.NewGoogleAuth()
-       code, err := gAuth.GetCode(user.Secret)
-
-       if err != nil {
-               pErr = pb.ErrorInternalErr(err.Error())
-               return
-       }
-
-       if req.Code != code {
-               pErr = pb.ErrorCodeNotMatch(pkg.ErrGoogleCode)
-               return
-       }
+       //gAuth := util.NewGoogleAuth()
+       //code, err := gAuth.GetCode(user.Secret)
+       //
+       //if err != nil {
+       //      pErr = pb.ErrorInternalErr(err.Error())
+       //      return
+       //}
+       //
+       //if req.Code != code {
+       //      pErr = pb.ErrorCodeNotMatch(pkg.ErrGoogleCode)
+       //      return
+       //}

```


# 项目结构

1. 程序入库：app/admin/cmd/server/main.go
2. 配置文件：app/admin/configs/config.yaml
3. 配置对象：app/admin/internal/conf/conf.proto


1. 所有的SERVICE（SERVICE）都注册到SERVER（SERVER）中
2. SERVICE对象中引用USECASE对象（BIZ）
3. USECASE对象中引用DATA对象（DB）
```
sysApiRepo := data.NewSysApiRepo(dataData, logger)
sysApiUseCase := biz.NewSysApiUseCase(sysApiRepo, casbinRuleRepo, logger)
apiService := service.NewApiService(sysApiUseCase, logger, casbinRuleUseCase)
```