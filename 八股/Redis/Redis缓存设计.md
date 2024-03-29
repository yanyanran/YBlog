## Redis缓存设计

#### 缓存雪崩

大量数据同时过期，解决：

1、**将缓存失效时间随机打散：** 在原有失效时间上加一个随机值，这样每个缓存的过期时间都不重复也就降低了缓存集体失效的概率

2、**互斥锁**：如果发现访问数据不在Redis里就加个互斥锁（再设个超时时间），保证同一时间内只有一个请求来构建缓存，缓存构建完成后再释放锁。

3、**后台更新缓存**： 后台线程不仅负责定时更新缓存，而且也负责**频繁地检测缓存是否有效**，检测到缓存失效就马上从数据库读取数据并更新到缓存；或者是在业务线程发现缓存数据失效后，通过**消息队列发送一条消息通知后台线程更新缓存**，后台线程收到后在更新缓存前可以判断缓存是否存在，存在就不执行更新缓存操作；不存在就读取数据库数据并加载到缓存

#### 缓存击穿

某热点数据过期且有大量请求访问该热点数据，解决：

1、**互斥锁**：保证同一时间只有一个业务线程请求缓存，未能获取互斥锁的请求，要么等待锁释放后重新读取缓存，要么就返回空或者默认值

2、不给热点数据设过期时间，由后台异步更新缓存，或者在热点数据准备要过期前，提前通知后台线程更新缓存以及重新设置过期时间

#### 缓存穿透

访问的数据既不在缓存也不在数据库中（非法数据），解决：

1、**非法请求限制**：在API入口处判断求请求参数是否合理

2、**设置空值或者默认值**：针对查询数据在缓存中设一个空值或者默认值，后续请求就可从缓存中读取到空值或者默认值返回给应用而不会继续查数据库

3、**布隆过滤器快速判断数据是否存在**：在写入数据库数据时使用布隆过滤器做个标记，用户请求到来时，业务线程确认缓存失效后，可通过查询布隆过滤器快速判断数据是否存在，即使发生了缓存穿透，大量请求只会查Redis和布隆过滤器而不会查数据库



#### 缓存更新策略

1、**旁路缓存策略**

在更新数据时，不更新缓存而是删除缓存中的数据。读数据时发现缓存中没了数据后，再从数据库中读数据更新到缓存中。又可分为「读策略」和「写策略」。

**写策略：**先更新数据库数据再删除缓存中的数据

**读策略：**如果读的数据命中了缓存则直接返回数据；如果读取数据没命中缓存则从数据库中读数据，然后将数据写到缓存并返回给用户

旁路缓存适合**读多写少**场景，不适合写多场景，因为写入比较频繁时缓存中数据会被频繁清理，会对缓存命中率有影响。（加锁/给缓存加个较短过期时间）

2、**读穿/写穿策略**

应用程序只和缓存交互，不再和数据库交互。

**读穿**：先查缓存中数据是否存在，存在直接返回，不存在则由缓存组件负责从数据库查询数据

**写穿**：有数据更新时先查要写入的数据在缓存中是否已存在，存在则更新缓存中的数据并由缓存组件同步更新到数据库中；不存在则直接更新数据库然后返回

3、**写回策略**

更新数据时只更新缓存，同时将缓存数据设置为脏然后立马返回，不会更新数据库。对于数据库的更新会通过批量异步更新的方式进行，适合**写多**场景。但数据不是强一致性的且会有数据丢失的风险