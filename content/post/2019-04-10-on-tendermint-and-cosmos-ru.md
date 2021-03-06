+++
author = ""
comments = true	# set false to hide Disqus
date = "2019-04-10T09:18:59+02:00"
draft = false
image = "images/posts/2019-04-15-basicblockradio-68-corrections/bbr.png"
menu = ""		# set "main" to add this content to the main menu
share = true	# set false to hide share buttons
slug = "on-tendermint-and-cosmos-ru"
tags = ["tendermint","cosmos"]
title = "Коротко о Tendermint и Cosmos Network"
+++

Ниже приведен текст заметки, которая была написана при подготовке к участию в
подкасте ["Базовый Блок"](https://basicblockradio.com/). Это не в коем случае
не исчерпывающий источник, а лишь короткий обзор включающий плюсы и минусы
Tendermint, Cosmos. Также упоминается проект Polkadot.

Выпуск со мной можно послушать [здесь](https://basicblockradio.com/e068/).

<!--more-->

## Tendermint

Tendermint это алгоритм консенсуса устойчивый к византийским падениям.
Византийские падения для тех кто не знает, это какие-то злоумышленные действия.
Алгоритм был придуман в 2014 году Джае Квоном, который был озабочен проблемой
высокого энергопотребления сети Bitcoin’а. В отличие от Nakamoto консенсуса,
где выбирается цепочка с [самым большым количеством
работы](https://medium.com/coinmonks/a-primer-on-blockchain-design-89605b287a5a),
в Tendermint выбирается цепочка где за блок проголосовало 2/3 участников сети.
Это упрощенная версия. Здесь я использую слово блок для понимания, на самом
деле это может быть чем угодно (any value).

В чем принципиальные отличия:

- выбор делается в пользу безопасности (safety) нежели доступности
  (availability) как в сети Bitcoin. Это значит, если 2/3 голосов не
  набирается, консенсус становится на паузу и ждет. Для более осведомленных
  товарищей, алгоритм безопасен в условиях асинхронной коммуникации и живуч?
  (т.е блоки создаются) в условиях частичной синхронности сети (safe in async
  network and requires partially synchronous network for liveness).
- Но если 2/3 набирается, блок можно считать окончательным. Т.е тебе не нужно
  ждать 10 или 100 блоков (probabilistic finality), чтобы убедиться что
  транзакция не будет откачена.
- Нет гонки за создание блока (энергозатраты минимальны). В данный момент в
  качестве алгоритма выбора ноды которая будет создавать блок используется
  циклический перебор взвешенный согласно доле криптовалюты. Это необязательно
  криптовалюты должна быть, могут быть просто попугаи. Т.е если у ноды A 100
  попугаев, у ноды B и C по 10 попугаев, то A будет чаще назначаться создателем
  блока.
  [спецификация](https://tendermint.com/docs/spec/reactors/consensus/proposer-selection.html#requirements-for-proposer-selection)

Для более подробного ознакомления читай [Tendermint Explained — Bringing
BFT-based PoS to the Public Blockchain
Domain](https://blog.cosmos.network/tendermint-explained-bringing-bft-based-pos-to-the-public-blockchain-domain-f22e274a0fdb)

Tendermint относится к семейству [BFT
алгоритмов](https://hackernoon.com/a-hitchhikers-guide-to-consensus-algorithms-d81aae3eb0e3).
Другие реализации - это
[HoneyBadgerBFT](https://twitter.com/zmanian/status/1089618263674212353),
Hyperledger (PBFT).

Раз это подкаст без буллшита :) расскажу про отрицательные стороны.

В чем минусы алгоритма:

1. Если нода, которая должна создать блок, его не создает, мы ждем некоторое
   время перед тем как начать новый раунд.
2. Алгоритм плохо масштабируется так как за каждый блок должно проголосовать
   2/3 сети. Редко какой BFT алгоритм может работать с 1000 и более нодами.

В чем плюсы:

1. Прост в понимании (проще чем PBFT)
2. Довольно быстрый
3. Блок не может быть отменен после того как 2/3 проголосовало за него и закоммитило (instant finality)

Когда люди говорят о Tendermint, чаще всего речь идет о утилите
https://github.com/tendermint/tendermint/ включающей вышеупомянутый алгоритм.
Помимо нее, она также включает p2p сеть, хранилище незакоммиченных транзакций
(mempool), RPC и интерфейс ABCI. ABCI - интерфейс (схема в protobuf по сути),
который позволяет разделить консенсус и логику приложения. То есть ты как
разработчик пишешь приложение на любом языке, Tendermint заботиться обо всем
остальном. **Это очень крутая концепция**.

В чем инновация Tendermint:

1. Практическая реализация! https://github.com/tendermint/tendermint/ уже
   существовавших концепций и новых наработок
2. ABCI

В будущем планируется предоставить приложению возможность менять алгоритм
выбора ноды создателя блока, совмещенные подписи (BLS) и многое другое.

Чтобы узнать больше, смотрите https://github.com/tendermint/awesome.

## Cosmos Network

Космос призван решить проблему отсутствия коммуникации между независимыми
блокчейнами. Делает он это посредством нового протокола (IBC). Вроде TCP/IP,
только для блокчейнов. Кто (блокчейн А) отправляет кому (блокчейн B) что (100
попугаев) + заголовок блока + доказательство merkle перехода в новый стейт.

IBC данный момент находится в разработке - https://github.com/cosmos/ics

Чтобы следить за состоянием балансов блокчейнов и уменьшить количество связей
между ними, были придуманы хабы (аналог свича в Интернете). Собственно недавно
был запущен первый хаб (Cosmos Hub). Чтобы вы понимали, “аналогичные” вещи есть
и в Polkadot (там это называется Relay), Ethereum 2.0 (beacon chain), Near
protocol (beacon chain).

Cosmos также предоставляет [SDK](https://cosmos.network/sdk), написанный как и
Tendermint на языке Go, для разработки приложений. В него включены модуль
переводов, модуль управления (governance), IBC модуль чтобы взаимодействовать с
другими блокчейнами. Позже будут добавлены другие модули (например NFT).

Аналогично с Tendermint, приведу основные минусы:

1. Если ты посылаешь IBC транзакцию из блокчейна А в блокчейн B через хаб, ты
   должен доверять валидаторам (нодам которые участвуют в консенсусе) блокчейна
   A, хаба и блокчейна B.
   https://medium.com/nearprotocol/unsolved-problems-in-blockchain-sharding-2327d6517f43
   Это одна из неразрешенных проблем. Я предлагаю прочитать статью Александра
   из near protocol, которая будет прикреплена в ссылках. Если коротко, то
   лучшее что пока придумали это переодически делать копии блоков (snapshots) и
   иметь процессы (fisherman; рыбак) которые следят за корректностью. Если
   рыбак замечает неправильный переход в новое состояние, он посылает
   challenge. Если challenge подтверждается, то реле (или бекон цепочка)
   откатывается до последней копии (snapshot) и blockchain B тоже откатывается.
   Плюс использовать erasure codes (коды стирания) для решения проблемы
   доступности блоков. В космосе можно создать хаб который будет требовать зоны
   использовать erasure codes и только принимать IBC транзакции с
   доказательствами доступности. Вопрос в приоритетах. О Polkadot, Near
   protocol и Ethereum 2.0 можно поговорить отдельно. "cosmos purposely focuses
   on enabling chains to economically integrate without politically
   integrating. $BNB holders can remain only broadly disinterested in the
   preferences of $ATOM holders. Governance decisions by $DOT holders affect
   every parachain"(https://twitter.com/zmanian/status/1113444044435099648)
2. Требуется instant finality (не минус, скорее требование). В случае с
   Биткоином например придется написать адаптер который ждет 6/10 блоков перед
   тем как считать транзакцию закоммиченной.

В чем плюсы:

1. Любой блокчейн может присоединиться.
2. Единый простой протокол для общения.
3. Гибкая структура. Например, блокчейн A может общаться напрямую с блокчейном B.
4. Пользователи не должны платить за общую безопасность сети. О чем я? Когда вы
   переводите средства в Ethereum 1.0 / Bitcoin, вы платите единую комиссию
   валидаторам. Но возможно (в случае игры или котиков например), вам не нужен
   такой высокий уровень безопасности. В космосе у игры будет свой возможно
   меньший по размеру набор валидаторов, нежели у хаба или блокчейна который
   занимается переводами.

В чем инновация Cosmos:

1. IBC
2. Сообщество профессиональных валидаторов

### Cosmos vs Polkadot

- [Техническое сравнение](https://forum.cosmos.network/t/polkadot-vs-cosmos/1397/2)
- [Более абстрактно и с
  картинками](https://tokeneconomy.co/the-state-of-crypto-interoperability-explained-in-pictures-654cfe4cc167)
- [Одно из опасений насчет Polkadot](https://twitter.com/jleni/status/1113464955020480512)
- Cosmos написан на Go (простой для изучения, в меру производительный),
  Polkadot - на Rust (высокопроизводительный, но порог входа в язык намного
  выше)
- на длинной дистанции проекты реализуют лучшие части друг друга.

Чтобы узнать больше, смотрите https://cosmos.network/ и https://github.com/cosmos/awesome.
