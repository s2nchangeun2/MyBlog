# Unity ECS (Entity-Component-System): 성능 최적화와 확장성의 핵심

## 📌 개요
Unity의 **ECS (Entity-Component-System)** 아키텍처는 게임 성능 최적화와 확장성을 목표로 설계되었다. ECS는 게임 오브젝트를 관리하는 기존의 방법과는 다른 접근 방식을 제공하며, 성능을 극대화할 수 있다.

---

## 🛠️ 1. ECS 아키텍처의 기본 개념

### 🔹 1.1 Entity (엔티티)

**엔티티**는 게임 내에서 **개별 객체**를 나타내며, 단순히 **ID**만 가지고 있는 객체이다. 엔티티는 데이터나 행동을 가지지 않으며, 컴포넌트들을 통해 기능을 추가할 수 있다. 엔티티는 **상태를 가진 데이터**가 아니라 **구성 요소**로서, 시스템에 의해 처리된다.

- 예시: 몬스터 엔티티는 `Health`, `Transform`, `Velocity` 컴포넌트를 가질 수 있다.

### 🔹 1.2 Component (컴포넌트)

**컴포넌트**는 엔티티의 **상태**나 **속성**을 저장하는 **데이터 구조**이다. 컴포넌트는 독립적이고 로직을 포함하지 않으며, 오직 **값**만을 저장한다. 컴포넌트는 일반적으로 **C# 구조체**로 작성되며, 시스템에서 이 데이터를 처리한다.

- 예시: `Transform` 컴포넌트는 엔티티의 위치, 회전, 크기를 저장합니다. `Velocity` 컴포넌트는 엔티티의 이동 속도를 나타낸다.

### 🔹 1.3 System (시스템)

**시스템**은 컴포넌트를 처리하는 **로직**을 담고 있는 부분이다. 시스템은 특정 조건에 맞는 엔티티를 선택하고, 이들의 컴포넌트를 읽거나 수정한다. 시스템은 게임 로직을 구현하는 핵심적인 부분이다.

- 예시: `MonsterMoveSystem`은 `Transform`과 `Velocity` 컴포넌트를 처리하여 몬스터의 위치를 업데이트한다.

---

## 🛠️ 2. 아키타입과 청크

### 🔹 2.1 Archetype (아키타입)

**아키타입**은 동일한 컴포넌트 타입을 가진 엔티티들을 묶는 구조이다. 즉, 아키타입은 특정 컴포넌트 세트를 가진 엔티티들을 **효율적으로 관리**하는 방법을 제공한다. 아키타입에 속하는 엔티티들은 **청크(Chunk)**라는 메모리 블록에 저장된다.

### 🔹 2.2 Chunk (청크)

**청크**는 엔티티와 그들의 컴포넌트를 메모리 상에서 효율적으로 저장하는 단위이다. 각 청크는 엔티티와 그에 해당하는 컴포넌트들을 배열로 저장하며, 이를 통해 **캐시 효율성을 높이고 성능을 최적화**한다.

- 예시: `Health`, `Transform`, `Velocity` 컴포넌트를 가진 모든 엔티티는 하나의 청크에 저장된다.

---

## 🛠️ 3. 효율적인 데이터 처리

### 🔹 3.1 쿼리(Query) 처리

ECS에서 **쿼리**는 특정 조건을 만족하는 엔티티들을 찾는 방법이다. 쿼리를 사용하면, 엔티티들이 포함된 **아키타입**을 기준으로 데이터를 효율적으로 검색할 수 있다.

- 예시: `Monster` 컴포넌트를 가진 모든 엔티티를 찾는 쿼리.

```csharp
var query = SystemAPI.Query<RefRW<LocalTransform>, RefRO<Velocity>>()
                     .WithAll<Monster>();

```

## 🔹 3.2 반복(Iterating) 처리

ECS는 반복을 통해 데이터를 순차적으로 처리한다. 이는 메모리 접근 패턴을 최적화하고 캐시 효율성을 극대화하는 방식으로 동작한다.

---

## 🛠️ 4. 멀티스레딩과 잡 시스템

### 🔹 4.1 멀티스레딩 지원

Unity ECS는 **멀티스레딩**을 지원하여 성능을 최적화할 수 있다. 이를 통해 게임의 여러 로직을 병렬로 처리할 수 있다. **잡 시스템**은 이러한 멀티스레딩을 활용하여 성능을 높인다.

**예시**: `IJobEntity`를 사용하여 엔티티 데이터를 병렬로 처리할 수 있다.

```csharp
public struct MonsterMoveJob : IJobEntity
{
    public float DeltaTime;

    public void Execute(ref LocalTransform transform, in Velocity velocity)
    {
        transform.Position += velocity.Value * DeltaTime;
    }
}
```

### 🔹 4.2 잡 스케줄링

잡을 **스케줄링**하여 병렬로 처리할 수 있으며, `ScheduleParallel` 메서드를 사용하여 여러 작업을 동시에 실행할 수 있다.

```csharp
var job = new MonsterMoveJob { DeltaTime = SystemAPI.Time.DeltaTime };
job.ScheduleParallel();
```

## 🔹 5. 하위 씬과 베이킹 시스템

### ✅ 5.1 하위 씬(Subscene)

**하위 씬**은 Unity ECS에서 게임 오브젝트와 ECS 엔티티를 분리하는 방식이다. 하위 씬을 통해 게임 오브젝트를 ECS 엔티티로 변환하여 게임 성능을 최적화한다.

### ✅ 5.2 베이킹(Baking)

**베이킹**은 게임 오브젝트의 데이터를 ECS 엔티티로 변환하는 과정이다. 이 과정을 통해 성능이 중요한 부분을 최적화할 수 있다.

```csharp
class EnergyShieldAuthoring : MonoBehaviour
{
    public int MaxHitPoints;
    public float RechargeDelay;
    public float RechargeRate;
}

class Baker : Baker<EnergyShieldAuthoring>
{
    public override void Bake(EnergyShieldAuthoring authoring)
    {
        var entity = GetEntity(TransformUsageFlags.None);
        AddComponent(entity, new EnergyShield());
    }
}
```

## 🛠️ 6. 스트리밍을 통한 효율적인 메모리 관리

### 🔹 6.1 대규모 환경에서의 스트리밍

Unity ECS는 대규모 환경에서 효율적인 메모리 관리를 지원한다. 스트리밍 시스템을 활용하여 필요한 데이터를 동적으로 로드하고 언로드할 수 있다.

---

## 🔚 결론

- ✔️ Unity의 ECS는 게임 성능 최적화에 매우 유용한 아키텍처다. 데이터와 로직을 효율적으로 분리하고, 멀티스레딩과 잡 시스템을 활용하여 성능을 극대화할 수 있다. ECS는 대규모 게임 개발과 성능이 중요한 게임에서 특히 유용하게 사용될 수 있으며, 이를 통해 게임 개발자들은 더 나은 성능을 가진 게임을 만들 수 있다.


📚 **유니티 공식 문서를 참고하여 작성하였습니다.🚀**  
