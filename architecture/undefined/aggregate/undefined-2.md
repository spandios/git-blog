# 애그리거트를 팩토리로 사용하기

상품 등록 전 상점 계정이 차단 상태가 아닌 경우를 체크하는 로직이 있다고 가정해보자.&#x20;

```java
public ProductId registerNewProduct(newProductRequest req){
     Store store = storeRepository.findById(req.storeId);   
     if(store.isBlocked){
          throw new StoreBlockedException();
     }
     Product product = new Product(id, store.id , ~ )
     productRepository.save(product);
     return product.id;
}
```

이 코드는 검증과 생성이 분리 되어 있으며, 도메인 로직에서 처리가 아니라 응용 서비스가 처리하고 있다.&#x20;

문제는 이런 코드는 응집도가 떨어지고 중복이 생길 수 있다.&#x20;

다음과 같이 Store 도메인에서 검증과 도메인 생성을 처리하는 것으로 수정해보자.

```java
class Store{
    public Product createProdut(~~) { 
        if(isBlokced()) throw new StoreBlockedException();
        return new Product(~~);
    }
}
```

이제 응용 서비스가 검증과 생성 처리 직접 하지 않고 도메인 서비스에 위임하기 때문에 만약 검증과 생성 처리에 변경이 있다 해도 도메인 서비스 코드만 변경하면 된다.&#x20;

도메인의 응집도가 높아지면서 결합도가 낮아진 것이다.&#x20;

어떤 데이터를 이용해서 다른 애그리거트를 생성해야한다면 애그리거트에 위에 메서드처럼 팩토리 메서드를 구현하는 것도 고려해보자.

만약 너무 많은 정보를 알아야한다면 다른 팩토리에 위임하는 방법도 있으니 참고하자.

