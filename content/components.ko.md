### 프로바이더

기본적으로 서비스, 레파지토리, 펙토리, 헬퍼 등등 거의 모든 것은 프로바이더로 생각할 수 있고 종속성을 **주입** 할 수 있습니다. 즉, 서로 다양한 관계를 만들 수 있습니다. 사실, 프로바이더는 `@Injectable()` 데코레이터로 주석이 달린 단순한 클래스에 지나지 않습니다.

<figure><img src="/assets/Components_1.png" /></figure>

이전 챕터에서, 간단한 `CatsController`를 만들었습니다. 컨트롤러는 HTTP 요청을 처리해야하고 **프로바이더**에게 좀 더 복잡한 작업을 위임해야 합니다. 프로바이더는 `@Injectable()` 데코레이터를 사용한 일반 자바스크립트 클래스이다.

> info **Hint** 네스트는 다양한 방법으로 종속성을 디자인하고 구성할 수 있기 때문에 **SOLID** 원칙을 따르는 것을 강하게 권장합니다.

#### 서비스

`CatsService` 프로바이더를 만드는 것으로 시작해봅시다.

```typescript
@@filename(cats.service)
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
@@switch
import { Injectable } from '@nestjs/common';

@Injectable()
export class CatsService {
  constructor() {
    this.cats = [];
  }

  create(cat) {
    this.cats.push(cat);
  }

  findAll() {
    return this.cats;
  }
}
```

> info **Hint** CLI를 통해서 서비스를 만들려면 간단하게 `$ nest g service cats` 명령을 수행하세요.

`CatsService`는 하나의 프로퍼티와 두개의 메소드로 구성되어 있는 기본 클래스입니다. 다른 점이 있다면 `@Injectable()` 데코레이터를 사용했습니다. `@Injectable()`은 메타데이터를 추가하기 때문에 네스트는 이 클래스가 네스트 클래스임을 알 수 있습니다. 위에서 아래의 `Cat` 인터페이스를 사용했습니다:

```typescript
export interface Cat {
  name: string;
  age: number;
  breed: string;
}
```

서비스 클래스를 작성하였으니 `CatsController`에서 사용해 봅시다:

```typescript
@@filename(cats.controller)
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private readonly catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
@@switch
import { Controller, Get, Post, Body, Bind, Dependencies } from '@nestjs/common';
import { CatsService } from './cats.service';

@Controller('cats')
@Dependencies(CatsService)
export class CatsController {
  constructor(catsService) {
    this.catsService = catsService;
  }

  @Post()
  @Bind(Body())
  async create(createCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll() {
    return this.catsService.findAll();
  }
}
```

`CatsService`는 클래스 생성자를 통해 주입되었습니다. `private readonly` 단축 구문을 두려워 하지 마세요. 이것은 `catService` 맴버 변수를 그 자리에서 즉시 생성하고 초기화 했다는 것을 의미합니다.

#### 의존성 주입

네스트는 **의존성 주입** 으로 알려진 강력한 디자인 패턴을 중심으로 만들어졌습니다. 아래의 [앵귤러](https://angular.io/guide/dependency-injection) 공식 문서에 있는 이 컨셉에 대한 좋은 글을 읽으시길 추천합니다.

네스트에서 타입스크립트의 타입을 통해 프로바이더들을 컨트롤러의 생성자로 전달하거나 명시된 프로퍼티에 할당되기 때문에 의존성을 관리하는 것은 매우 쉽습니다:

```typescript
constructor(private readonly catsService: CatsService) {}
```

#### 스코프

각각의 프로바이더들은 어플리케이션 생명주기에 의존하는 생명주기를 가집니다. 한번 어플리케이션이 실행되면, 주어진 모든 프로바이더들은 인스턴스화 됩니다. 비슷하게 어플리케이션이 종료될 때 모든 프로바이더들은 소멸됩니다. 그러나 프로바이더들의 생명주기를 요청 단위 스코프로 처리할 수 있습니다. 관련된 내용은 [여기](/fundamentals/scopes) 에서 확인할 수 있습니다.

#### 커스텀 프로바이더

네스트에서는 IoC 컨테이너를 통해 프로바이더들간의 관계를 해결할 수 있습니다. 이것은 앞에서 설명한 것 보다 훨씬 강력합니다. `@Injectable()` 데코레이터는 프로바이더를 정의하는데 필수는 아닙니다. 그것을 대신해서 값, 클래스, 팩토리 등을 사용할 수 있습니다. 보다 자세한 내용은 [여기](/fundamentals/dependency-injection)를 확인하세요.

#### 선택적 프로바이더

때에 따라 처리될 필요가 없는 관계가 있을 수 있습니다. 예를 들어 어떤 클래스가 **설정 객체**를 의존하고 있고 값이 전달되지 않으면 디폴트 값이 사용되야 합니다. 이런 상황에서 설정 프로바이더가 없더라도 오류가 발생하지 않으므로 그 관계는 선택적일 수 있습니다.

프로바이더가 주입되지 않아도 된다면 `constructor`안에서 `Optional()` 데코레이터를 사용하세요.

```typescript
import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  constructor(
    @Optional() @Inject('HTTP_OPTIONS') private readonly httpClient: T,
  ) {}
}
```

#### 프로퍼티 주입

몇몇 특별한 상황에서, 프로퍼티 주입은 유용할 수 있습니다. 예를 들어 최상위 클래스에서 하나 이상의 프로바이더를 의존하고 있다면 하위 클래스의 생성자에서 `super()`를 호출해서 그것들을 전달하는 것은 아주 성가실 수 있습니다. 그래서 프로퍼티 레벨에서 `@Inject()` 데코레이터를 사용해서 그것을 피할 수 있습니다.

```typescript
import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  @Inject('HTTP_OPTIONS')
  private readonly httpClient: T;
}
```

> warning **Warning** 만약 클래스가 어떠한 프로바이더도 상속받지 않는다면 **생성자 주입**을 사용하는 것을 항상 선호하세요.

#### 프로바이더 등록

마지막으로 모듈에 `CatsService`의 존재를 알려야 합니다. 여기서는 `app.module.ts`파일에 `@Module()` 데코레이터안에 있는 `providers` 배열에 그 서비스를 추가함으로써 해결할 수 있습니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class ApplicationModule {}
```

그렇게 되면, 네스트는 `CatsController` 클래스의 의존성을 해결할 수 있을 것입니다. 현재 디렉토리 구조는 다음과 같습니다:

<div class="file-tree">
<div class="item">src</div>
<div class="children">
<div class="item">cats</div>
<div class="children">
<div class="item">dto</div>
<div class="children">
<div class="item">create-cat.dto.ts</div>
</div>
<div class="item">interfaces</div>
<div class="children">
<div class="item">cat.interface.ts</div>
</div>
<div class="item">cats.service.ts</div>
<div class="item">cats.controller.ts</div>
</div>
<div class="item">app.module.ts</div>
<div class="item">main.ts</div>
</div>
</div>
