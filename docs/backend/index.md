---
title: 后端
date: 2026-03-14
slug: backend
tags:
  - backend
status: published
visibility: public
summary: 后端设计、接口治理、数据与实现经验。
hide:
  - toc
description: "后端设计、接口治理、数据与实现经验。"
---
<div class="page-head">
  <p class="eyebrow">BACKEND</p>
  <h1>后端</h1>
  <p class="page-head__lead">
    面向真实交付场景的后端内容：接口设计、服务治理、数据存储和代码实现，
    以及那些只有上线之后才会暴露出来的维护成本。
  </p>
</div>

<div class="panel panel--list"><div class="log"><a class="log__row" href="../backend/wallet-billing-engine-redis-db-loki/"><time class="log__date" datetime="2026-05-15">05.15</time><div class="log__main"><p class="log__meta"><span class="log__cat">后端</span><span class="log__tags">backend / redis / billing</span></p><h3 class="log__title">钱包扣费引擎：Redis 管余额，DB 管账单，日志管追凶</h3><p class="log__summary">多连接时长计费不应该靠 tick 流水硬扛，而应该把实时余额、最终账单和审计日志拆成三套职责。</p></div><span class="log__read">10 MIN</span><span class="log__arrow" aria-hidden="true">↗</span></a></div></div>
