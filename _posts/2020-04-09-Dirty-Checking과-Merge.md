---
layout: post
title: "Dirty Checking과 Merge"
author: cupjoo
categories: [JPA]
image: assets/images/2020-04-09/1.png
---

관련 포스트 : [영속성 컨텍스트](https://cupjoo.github.io/영속성-컨텍스트)

---

사용자 페이지에서 정보 수정이 발생해 컨트롤러에 엔티티 폼이 넘어올 때, 엔티티 값 수정을 반영하려면 어떻게 해야 할까?

```java
// ItemController
public class ItemController {

    @PutMapping("/item/{id}/edit")
    public String updateItem(
        @PathVariable String itemId,
        @ModelAttribute("form") ItemForm form){

        Item item = Item.builder()
            .id(itemId)
            .name(form.getName())
            .price(form.getPrice())
            .stockQuantity(form.getStockQuantity)
            .build();
        itemService.update(item);
        return "redirect:/items/" + itemId;
    }
}
```

위에서 생성되는 item은 `준영속 엔티티`다. 준영속 엔티티는 영속성 엔티티였던 객체가 영속성을 잃어 영속성 컨텍스트에서 더 이상 관리되지 않는 엔티티를 말한다.

기존에 생성된 엔티티로부터 식별자 (id)를 받아 임의로 생성된 엔티티는 준영속 엔티티가 된다. 따라서 item은 `변경 감지 (Dirty Checking)`가 자동으로 발생하지 않는다. 따라서 우리는 준영속 엔티티를 수정하기 위해 **변경감지**와 **병합(merge)** 2가지 방법 중 하나를 사용해야 한다.

```java
// ItemService
public class ItemService {

    // Dirty Checking
    @Transactional
    public void update2(Item item){
        Item findItem = itemRepository.findById(item.getId());
        findItem.changePrice(item.getPrice());
    }

    // Merge
    @Transactional
    public void update(Item item){
        Item mergeItem = itemRepository.save(item)
    }
}
```

```java
// ItemRepository
public class ItemRepository {

    public void save(Item item){
        if(item.getId() == null){
            em.persist(item);
        } else {
            em.merge(item);
        }
    }
}
```

전자는 Controller로부터 생성된 Item 객체를 사용해 실제 Item 객체를 조회해온다. 그 뒤 가져온 item의 값을 변경해 변경 감지가 발생하게 한다.

후자는 바로 save를 시키고, repository에서 이미 존재하는 아이템인 경우 merge한다. merge의 내부 로직은 전자와 동일하게 해당 item을 가져온 뒤 변경 감지를 통해 값을 변경한다. 단, 모든 속성 값을 변경하기 때문에, 만약 item 속성 중 null 값이 있는 경우에도 그대로 반영이 되어 데이터 손실이 발생할 수 있다.

따라서 전자인 Dirty Checking이 더 선호되는 방식이다. 하지만 더 좋은 방식은 변경할 데이터만을 정확히 명시해 전달하는 것이다. 그러면 더 안전하게 변경 감지를 사용할 수 있게 된다.

```java
@Transactional
public void updatePrice(Long itemId, int price){
    Optional<Item> findItem = itemRepository.findById(itemId);
    findItem.ifPresent(s -> s.changePrice(price));
}
```

- Controller 계층에서 굳이 엔티티를 새롭게 생성하지 않는다.
- Service 계층에 식별자 (id)와 변경할 데이터만을 명확하게 전달한다.
- Service 계층에서 영속성 엔티티를 조회한 뒤, 데이터를 직접 변경한다.

## 참고 자료

> [자바 ORM 표준 JPA 프로그래밍](https://book.naver.com/bookdb/book_detail.nhn?bid=9252528)
