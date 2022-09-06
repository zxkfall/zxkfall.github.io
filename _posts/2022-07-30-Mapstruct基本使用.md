---
layout: post
title: "Mapstruct基本使用"
subtitle: Mapstruct basic using
categories: java
tags: [ java, mapstruct, bean]
banner: "/assets/images/cover/2022-07-30-universe.jpg"

---

在日常的项目开发过程中，数据的存储结构和我们想要得到的数据结构往往是不一样的，为了让数据可以更好的传输，我们往往需要对获得的原始数据进行一些封装，将封装好的数据进行传输，在Java中，大多数的工具类常常使用反射的方式对对象的属性进行提取，这样就会降低程序的运行速度。而Mapstruct则是根据接口生成对应的属性转换类。这样只会增加程序编译的时间，由于在程序运行过程中是调用的转换类对对象的属性进行转换，因此和反射相比，可以大大加快程序的运行速度。

<!--more-->

## 1. 基本使用

假设需要进行转换的类为

```java
@Data
public class Car {
    private String make;
    private int numberOfSeats;
    private CarType type;
 }
```

转换后的类为

```java
public class CarDto {
    private String make;
    private int seatCount;
    private CarType type;
}
```

如果要进行转换，需要定义一个接口来编写转换的逻辑

```java
@Mapper 
public interface CarMapper {
 
    CarMapper INSTANCE = Mappers.getMapper( CarMapper.class ); 
 
    @Mapping(source = "numberOfSeats", target = "seatCount")
    CarDto carToCarDto(Car car); 
}
```

使用

```java
@Test
public void shouldMapCarToDto() {
    //given
    Car car = new Car( "Morris", 5, CarType.SEDAN );
 
    //when
    CarDto carDto = CarMapper.INSTANCE.carToCarDto( car );
 
    //then
    assertThat( carDto ).isNotNull();
    assertThat( carDto.getMake() ).isEqualTo( "Morris" );
    assertThat( carDto.getSeatCount() ).isEqualTo( 5 );
    assertThat( carDto.getType() ).isEqualTo( "SEDAN" );
}
```

只要字段名一样，Mapstruct就会尝试进行转换，比如将枚举类的枚举类型转化为字符串，如果无法转换就会报错。

## 2. 在Spring中的使用

Mapstruct也可以使用依赖注入的方式进行使用，从而可以被Spring的容器所管理，具体的使用过程如下

```java
@Component
@AllArgsConstructor
public class GoodListConvert {
    private final GoodMapper goodMapper;

    @Named("getGoodResponse")
    public List<GoodResponse> getGoodResponse(Orders orders) {
        return orders
                .getGoodsList().stream()
                .map(goodMapper::goodsToGoodResponse)
                .collect(Collectors.toList());
    }

}
```





```java
@Mapper(componentModel = "spring", unmappedSourcePolicy = ReportingPolicy.IGNORE)
public interface GoodMapper {

    GoodResponse goodsToGoodResponse(Goods goods);
}
```





```java
@Mapper(componentModel = "spring",
        unmappedSourcePolicy = ReportingPolicy.IGNORE,
        uses = {GoodListConvert.class, DeliveryTypesConvert.class, TotalPriceConvert.class})
public interface OrderMapper {

    @Mapping(target = "totalPrice", qualifiedByName = "sumTotalPrice", source = ".")
    @Mapping(target = "goodsList", qualifiedByName = "getGoodResponse", source = ".")
    @Mapping(target = "deliveryTypes", qualifiedByName = "getDeliveryTypes", source = ".")
    OrderResponse ordersToOrderResponse(Orders orders);

}
```





参考资料

1. [Mapstruct官网](https://mapstruct.org/)
