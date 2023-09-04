---
title: "[JPA] 준영속 엔티티와 수정 방법"

categories:
  - spring
tags:
  - [spring, jpa]

toc: true
toc_sticky: true

date: 2023-09-04
last_modified_at: 2023-09-04
---


# 준영속 엔티티?

- 한 마디로 persis context 가 더 이상 관리하지 않는 엔티티를 말한다.
- 영속과 준영속 예시를 보며 비교해보자

## 영속 엔티티


```

  @Transactional
  public void cancleOrder(Long orderId) {

      //주문 엔티티 조회
      Order order = orderRepository.findOne(orderId);

      //주문 취소
      order.cancel();
  }

  ...

  //Order.class

  public void cancel() {
      if (delivery.getStatus() == DeliveryStatus.COMP) {
          throw new IllegalStateException("이미 배송완료된 상품은 취소가 불가능합니다.");
      }

      this.setStatus(OrderStatus.CANCEL);
      for (OrderItem orderItem : orderItems) {
          orderItem.cancle();
      }
  }

```

위 코드는 영속 엔티티에 대한 예시이다. order 객체는 @Transactional 을 통해 영속 컨텍스트 안에 존재하는 엔티티이다.
이후 order 객체의 속성 값을 변경하게 되면 업데이트 쿼리 없이도 DB 에 변경된 값이 자동으로 등록된다.
order 객체가 영속 컨텍스트에서 관리되기 때문이다. 그러면 영속 컨텍스트에서 관리되지 않는 준영속 엔티티는 무엇일까? 


## 준영속 엔티티

```
  @PostMapping("items/{itemId}/edit")
  public String updateItem(@ModelAttribute("form") BookForm form, @PathVariable String itemId) {

      Book book = new Book();

      book.setId(form.getId());
      book.setName(form.getName());
      book.setPrice(form.getPrice());
      book.setStockQuantity(form.getStockQuantity());
      book.setAuthor(form.getAuthor());
      book.setIsbn(form.getIsbn());

      itemService.saveItem(book);

      return "redirect:/items";
  }

```

위 코드는 특정 item 에 대한 수정 요청이 들어왔을 때 Book 이라는 새로운 엔티티를 만들어내고 form 에서 변경된 값들을 set 한 후에 saveItem 을 통해서 저장하는 방법이다. 
여기서 가장 중요한 부분은 아래와 같다.

>       book.setId(form.getId());

Id 값은 DB 에 저장될 때 만들어지는 것이므로 Id 가 이미 존재한다는 것은 해당 객체가 이미 DB 에 존재한다는 것이다. 즉, 핵심은 식별자를 기준으로 영속 상태가 되어서 DB 에 저장된 적이 있는가로 준영속을 판단할 수 있다.

따라서 식별자를 기준으로 한 번 영속 상태가 된 엔티티가 더 이상 영속성 컨텍스트에서 관리되지 않으면 모두 준영속 상태이다.

영속성 컨텍스트에서 관리되지 않기 위해 em.detach() 를 직접 할 수도 있고, 위와 같이 새로운 엔티티에 기존 Id 값을 부여할 수 있다. 


# 준영속 엔티티 수정하기

준영속 엔티티를 DB 까지 수정하기 위해서는 병합 (merge) 과 변경 감지 기능 (dirty check) 이 있다.

## 병합 

```
  @PostMapping("items/{itemId}/edit")
  public String updateItem(@ModelAttribute("form") BookForm form, @PathVariable String itemId) {

      Book book = new Book();

      book.setId(form.getId());
      book.setName(form.getName());
      book.setPrice(form.getPrice());
      book.setStockQuantity(form.getStockQuantity());
      book.setAuthor(form.getAuthor());
      book.setIsbn(form.getIsbn());

      itemService.saveItem(book);

      return "redirect:/items";
  }

```

위의 코드를 다시 살펴보자. 해당 코드는 기존에 DB 에 저장되어있는 객체에 대해서 수정된 사항을 변경해서 저장하는 코드이다.
book 객체는 준영속 엔티티로 속성 값들을 새롭게 set 한다고해서 DB 와 연동되지 않는다.
따라서 DB 에 저장해주는 과정이 필요하고 이 것이 ItemService 의 saveItem 메서드이다. 이에 대해 자세히 알아보자

```
//ItemServie.class


    @Transactional
    public void saveItem(Item item) {
        itemRepository.save(item);
    }


//ItemRepository.class


    public void save(Item item) {
        if (item.getId() == null) {
            //새로운 아이템일 경우 id 가 null
            em.persist(item);
        } else {
            em.merge(item);
        }
    }

```

결국 book 객체는 em.merge() 를 통해 DB 에 저장된다. 

![image](https://github.com/min9805/min9805.github.io/assets/56664567/9fe64c7b-6539-48df-bfcc-23803491aaa6)

위 그림은 병합 동작 방식을 나타낸다.

1. 가장 먼저 파라미터로 넘어온 준영속 엔티티의 식별자 값 (ID) 로 1차 캐시, DB 에서 조회한다.
2. 조회한 영속 엔티티에 준영속 엔티티 값들을 채워 넣는다.
3. 영속 상태인 엔티티를 반환한다.

---
### 단점

하지만 병합은 치명적인 단점이 존재한다. 

```
  @PostMapping("items/{itemId}/edit")
  public String updateItem(@ModelAttribute("form") BookForm form, @PathVariable String itemId) {

      Book book = new Book();

      book.setId(form.getId());
      book.setName(form.getName());
      // book.setPrice(form.getPrice());
      book.setStockQuantity(form.getStockQuantity());
      book.setAuthor(form.getAuthor());
      book.setIsbn(form.getIsbn());

      itemService.saveItem(book);

      return "redirect:/items";
  }

```

위 코드에서 만약 setPrice 가 존재하지 않는다면 기존의 price 가 저장되는 것이 아니라 null 값이 저장된다!


## 변경 감지 기능

병합의 치명적인 단점으로 변경 감지 기능이 추천된다. 

```
    @PostMapping("items/{itemId}/edit")
    public String updateItem(@ModelAttribute("form") BookForm form, @PathVariable String itemId) {

        itemService.updateItem(form.getId(), form.getName(), form.getPrice(), form.getStockQuantity());

        return "redirect:/items";
    }


//ItemService.class

    @Transactional
    public void updateItem(Long itemId, String name, int price, int stockQuantity) {
        Item findItem = itemRepository.findOne(itemId);
        findItem.setPrice(price);
        findItem.setName(name);
        findItem.setStockQuantity(stockQuantity);
    }

```

이는 @Transactional 안에서 조회를 통해 영속 엔티티를 그대로 가져온다.
해당 Item 은 영속 컨텍스트에 의해 관리되기 때문에 변경되는 값들이 바로 DB 와 연동된다.

# etc

- set 으로 변경하는 것은 설명일 뿐 Setter 자체가 권장되지 않는다.
- updateItem 에서 3번에 나누어서 update 하지 않고 하나의 change 메서드를 통해 변경하는 것이 옳다. 