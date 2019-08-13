### 어플리케이션 컨텍스트

네스트 어플리케이션을 활용하여 웹 앱, 마이크로서비스 또는 네스트 **어플리케이션 컨텍스트**를 만들 수 있습니다. 네스트 컨텍스트는 네스트 컨테이너를 감싸고 있고 모든 인스턴스화 된 클래스들을 담고 있습니다. 어플리케이션 객체를 사용해서 불러온 모듈안에 존재하는 인스턴스를 가져올 수 있으며 **CRON**작업을 포함하여 어디서든 네스트 프레임워크를 활용할 수 있으며 그 위에서 **CLI**를 구축할 수 도 있습니다.

#### 시작하기

네스트 어플리케이션 컨텍스트를 생성하기 위해 다음의 문법을 사용하세요:

```typescript
@@filename()
async function bootstrap() {
  const app = await NestFactory.createApplicationContext(ApplicationModule);
  // application logic...
}
bootstrap();
```

이후에 네스트 어플리케이션에 등록된 어떠한 인스턴스도 가져올 수 있습니다. `TasksModule`안에 `TasksService`가 있다고 상상해 봅시다, 이 클래스는 CRON 작업 내에서 호출하려하는 메소드들의 집합을 제공합니다.

```typescript
@@filename()
const app = await NestFactory.create(ApplicationModule);
const tasksService = app.get(TasksService);
```

`TasksService` 인스턴스를 가져오기 위해서 `get()` 메소드를 사용합니다. `get()` 메소드는 각각 등록된 모듈에서 자동으로 검색하는 **쿼리**처럼 행동하기 때문에 모듈 전체를 거치지 않아도 됩니다. 만약 엄격한 컨텍스트 검사를 선호 한다면 `get()` 메소드의 두번째 인자로 전달되는 `strict:true` 옵션을 사용할 수 있습니다. 그러면 선택된 모듈로부터 특정 인스턴스를 가져오기 위해 모든 모듈을 거쳐야 한다.

```typescript
@@filename()
const app = await NestFactory.create(ApplicationModule);
const tasksService = app.select(TasksModule).get(TasksService, { strict: true });
```

<table>
  <tr>
    <td>
      <code>get()</code>
    </td>
    <td>
      어플리케이션 컨텍스트 안에서 유효한 컨트롤러 또는 프로바이더 (가드, 필터, 기타 등등을 포함)를 검색한다.
    </td>
  </tr>
  <tr>
    <td>
      <code>select()</code>
    </td>
    <td>
      선택된 모듈로 부터 특정 인스턴스를 가져오기 위해 모듈 그래프를 통해 탐색한다 (활성화된 엄격한 모드에서 함께 사용).
    </td>
  </tr>
</table>

> info **Hint** 엄격한 모드가 아닐 때, 루트 모듈이 디폴트로 선택된다. 다른 모듈을 선택하기 위해서는 모듈 전체를 단계별로 거칠 필요가 있다.
