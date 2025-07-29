
```table-of-contents
title: 
style: nestedList # TOC style (nestedList|nestedOrderedList|inlineFirstLevel)
minLevel: 0 # Include headings from the specified level
maxLevel: 0 # Include headings up to the specified level
include: 
exclude: 
includeLinks: true # Make headings clickable
hideWhenEmpty: false # Hide TOC if no headings are found
debugInConsole: false # Print debug info in Obsidian console
```
### 역할
- **복잡한 동적 쿼리** 처리 (MyBatis의 `<if>`, `<choose>` 태그 역할)
- **타입 세이프한 쿼리** 작성 (**컴파일 타임** 오류 체크)
- **조인이 많은 복합 쿼리** 처리

```java
@Repository
public class LiveRepositorySupport {
    private final JPAQueryFactory queryFactory;
    
    public LiveRepositorySupport(JPAQueryFactory queryFactory) {
        this.queryFactory = queryFactory;
    }
    
    // 복잡한 동적 쿼리들을 QueryDSL로 구현
    public List<Live> findLiveAuctionsWithConditions(SearchCondition condition) {
        return queryFactory
            .selectFrom(live)
            .where(
                titleContains(condition.getKeyword()),
                dateRange(condition.getStartDate(), condition.getEndDate()),
                statusEquals(condition.getStatus())
            )
            .orderBy(live.startDate.asc())
            .fetch();
    }
}
```