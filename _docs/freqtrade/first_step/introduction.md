---
title: FREQTRADE 처음 사용해보기
category: first_step
sub_category: freqtrade
order: 1
id: 5
# permalink: /NestJs/docker
# description: 도커를 이용한 Nestjs 배포 가이드
# parent: NESTJS
# parentUrl: /NestJs/docker/
# subparent: Configuration
# subparentUrl: /Prometheus/config/
# priority: 1
---

## FREQTRADE?

![FreqTrade](https://www.freqtrade.io/en/stable/assets/freqtrade_poweredby.svg) 

- Free & Opensource 가상화폐 거래 봇 [ 파이썬으로 작성됨 ]
- 전략을 직접 설정하고 실행해볼 수 있음.


## FIRST STEP

간단하게 도커를 이용해 테스트 해보기

```bash
docker run --rm freqtrade create-userdir --userdir user_data

docker run --rm freqtrade new-config --config user_data/config.json

```




### download data

> 바이낸스 / bittrex 등등 에서 거래 데이터를 쉽게 다운로드 받을 수 있다.

```bash

docker-compose run --rm freqtrade download-data --exchange binance --days 30 -t 1m -p ADA/USDT AXS/USDT BTC/USDT ETH/USDT

```

