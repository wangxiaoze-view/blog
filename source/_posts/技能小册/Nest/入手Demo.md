---
title: Nest-入手Demo
abbrlink: 89a2365c
date: 2024-11-08 09:05:01
series: Nest入门
categories:
  - 技能小册
tags:
  - Nest
---

简单了解一些`Nest`，结合之前我开发的一个小工具日志上报插件，做一个小应用;

这个应用是什么? 有这么一个场景，我在`web`页面使用了`日志上报插件`，但是插件的`dsn`地址是没有的，这个地址需要后端的一个接口地址，那我们用`Nest`模式一下这个接口地址；

接下来，实践一下：

## 初始化项目

首先，去[官网](https://docs.nestjs.com/)找一下安装命令

```shell
npm i -g @nestjs/cli
nest new project-name
```

安装成功之后，所有的代码都是`ts`编写的; 关于项目的目录结构，可以参考[官方文档](https://docs.nestjs.com/first-steps)有具体说明;

我们找到入口文件：`main.ts`, 其中`bootstrap()`就是启动函数了；

```ts
async function bootstrap() {
	// 这里使用的是Express, 可以使用Fastify
	const app = await NestFactory.create<NestExpressApplication>(AppModule, {
		// 如果上报方式使用sendBeacon， 那么就需要对请求头设置相关参数
		// 如果上报方式使用fetch, 那么就不需要设置相关参数
		rawBody: true,
	});
	// 根据请求的Content-Type来解析请求体, 如果是 sendBeacon 那么需要设置相关参数
	app.useBodyParser("text");

	// 设置跨域
	app.enableCors({
		origin: "*",
		credentials: true,
		methods: "GET,HEAD,PUT,PATCH,POST,DELETE",
		preflightContinue: false,
		optionsSuccessStatus: 204,
	});

	// 设置前缀, 可以设置也可以不设置 这里不设置
	// app.setGlobalPrefix("/api/v1");

	await app.listen(3000);
}

bootstrap();
```

修改之后，执行命令`npm run start`就可以了，不过这里可以使用`npm run start:dev`,只有文件变动了服务就会重新启动；就不需要每次修改之后在重启；

## 实现添加功能

### 数据库

添加数据肯定需要数据库哇，那我们先添加下载数据库插件，我这里使用的`Mongoose,mongodb`； 具体可查看[Nest Mongo](https://docs.nestjs.com/techniques/mongodb)

关于本地如何安装`mongodb`以及如何使用，可查看基础文档[MongoDb 教程](https://www.runoob.com/mongodb/mongodb-tutorial.html)

```shell
npm i --save @nestjs/mongoose mongoose
```

然后在`app.module.ts`中添加配置

```ts
import { MongooseModule } from '@nestjs/mongoose';
@Module({
  imports: [
    // 只需要在 imports 加入这一行即可;
    MongooseModule.forRoot('mongodb://localhost/nest'),
  ]
}
```

**保存之后重新启动，如果出现数据库一直尝试链接那么说明你本地没有正确安装`mongoose`服务,或者您的数据库链接可能不正确;**

### 添加一个日志模块

要想添加一个日志模块，你可以手动在`src`目录下添加目录为`logs`的文件夹，在文件夹下添加对应的文件；

不过我不建议你手动添加，不仅麻烦，而且也不好管理；

可以通过命令`nest` 去添加；想要知道命令有哪些，可以在`shell`工具中输入`nest`然后回车；

然后使用命令`nest g name dirFile`;

我们直接使用`nest g resource logs`, 回车选择对应的`Api模式`, 之后就会生成一个完整模块的目录了；使用命令添加还会自动在`app.modules.ts`添加对应的模块哦~

创建完之后，就可以在`src`目录下找到你的文件夹或者文件了

### 创建一个控制器

在`logs.controller.ts`中添加一个方法，用来添加`logs`数据

```ts
// 请求的前缀：https://xxx.com/logs
@Controller("logs")
export class LogsController {
	// 这里的LogsService就是服务层文件 logs.service.ts中的类
	constructor(private readonly logsService: LogsService) {}
	// https://xxx.com/logs/setLog
	@Post("setLog")
	// createLogDto 就是前端给后端的字段
	create(@Body() createLogDto: CreateLogDto) {
		return this.logsService.create(createLogDto);
	}
}
```

接着我们添加数据库需要的字段以及数据表;

### 添加数据表以及对应表结构

```ts
import { Prop, Schema, SchemaFactory } from "@nestjs/mongoose";
import { HydratedDocument } from "mongoose";

export type LogDocument = HydratedDocument<Log>;

@Schema()
export class Log {
	@Prop()
	name: string;
	@Prop()
	url: string;
	@Prop()
	// 创建时间
	createdAt: Date;
	// 更新时间
	updatedAt: Date;
	// ...
}
export const LogSchema = SchemaFactory.createForClass(Log);
```

大概的数据表结构就定好了；

我们在`logs.module.ts`引入我们创建的表

```ts
// 这个是你实际创建数据表的地址， LogSchema就是你导出的表
import { LogSchema } from "./schemas/log.schema";
@Module({
  imports: [
    MongooseModule.forFeature([
      {
        name: "Log",
        schema: LogSchema,
      }
    ])
  ]
}
```

### 添加服务层

在`logs.service.ts`中添加一个`create`方法

```ts
interface CreateLogDto {
	name: string;
	url: string;
}

@Injectable()
export class LogsService {
	constructor(
		// 这里是用到了数据库 这里的 @InjectModel("Log") Log要个上面的 MongooseModule.forFeature 中的name保持一致;
		//  Model<Log> 中Log就是你到处的表结构 class Log{}
		@InjectModel("Log") private readonly logModel: Model<Log>
	) {}

	// 这个方法就是 logs.controller.ts 中使用的 this.logsService.create
	// CreateLogDto 这个是为了检验参数的， 也就是接口 interface 你自己可以自定义
	create(createLogDto: CreateLogDto) {
		// 这里使用 Model<Log> 的 create 方法，具体mongodb如何自行百度
		return this.logModel.create(createLogDto);
	}
}
```

## 使用

可以使用`postman`或者`apipost`或者`apifox`测试； 具体工具如何使用，自行百度官网;

添加一个`post`接口，接口地址就是`localhost:3000/logs/setLog`, 参数就是上面自定义的接口`interface CreateLogDto`， 然后点击发送就可以了;

那么一个小应用就基本完成了;
