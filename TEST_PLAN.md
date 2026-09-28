# TEST_PLAN — bolt11/index.html

Декодер тестируется **вручную в браузере** (5 кнопок-примеров) + сверкой
ожидаемых значений ниже. Все валидные векторы — из BOLT11-спеки
(`lightning/bolts`, раздел Examples, только чтение).

## Прогон

```sh
cd bolt11
python3 -m http.server 8097
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8097/index.html
# ожидалось: 200
```

Открыть `http://localhost:8097/index.html`, нажимать кнопки 1–5, сверять.

## T1. lnbc без суммы (donation, payment_hash 000102…)

Инвойс:

```text
lnbc1pvjluezsp5zyg3zyg3zyg3zyg3zyg3zyg3zyg3zyg3zyg3zyg3zyg3zyg3zygspp5qqqsyqcyq5rqwzqfqqqsyqcyq5rqwzqfqqqsyqcyq5rqwzqfqypqdpl2pkx2ctnv5sxxmmwwd5kgetjypeh2ursdae8g6twvus8g6rfwvs8qun0dfjkxaq9qrsgq357wnc5r2ueh7ck6q93dj32dlqnls087fxdwk8qakdyafkq3yap9us6v52vjjsrvywa6rt52cm9r9zqt8r2t7mlcwspyetp5h2tztugp9lfyql
```

Ожидается: `hrp=lnbc`, сеть `mainnet`, сумма `(не указана)`,
`timestamp=1496314658` (`2017-06-01T…`),
`payment_hash=0001020304050607080900010203040506070809000102030405060708090102`,
`payment_secret=1111111111111111111111111111111111111111111111111111111111111111`,
`description=Please consider supporting this project`, `h=—`,
`expiry=3600` (по умолч.), `r-полей=0`, `signature present=да`.
Статус: PASS (сверено с разбором Breakdown в спеке).

## T2. lnbc2500u кофе, expiry 60

Инвойс:

```text
lnbc2500u1pvjluezsp5zyg3zyg3zyg3zyg3zyg3zyg3zyg3zyg3zyg3zyg3zyg3zyg3zygspp5qqqsyqcyq5rqwzqfqqqsyqcyq5rqwzqfqqqsyqcyq5rqwzqfqypqdq5xysxxatsyp3k7enxv4jsxqzpu9qrsgquk0rl77nj30yxdy8j9vdx85fkpmdla2087ne0xh8nhedh8w27kyke0lp53ut353s06fv3qfegext0eh0ymjpf39tuven09sam30g4vgpfna3rh
```

Ожидается: сумма `2500u` → `0.0025 BTC` = `250000 sats` = `250000000 msats`,
`description=1 cup coffee`, `expiry=60` (явный), тот же `payment_hash/secret`,
`signature present=да`. Статус: PASS.

## T3. lnbc20m hashed list (description_hash)

Инвойс:

```text
lnbc20m1pvjluezsp5zyg3zyg3zyg3zyg3zyg3zyg3zyg3zyg3zyg3zyg3zyg3zyg3zygspp5qqqsyqcyq5rqwzqfqqqsyqcyq5rqwzqfqqqsyqcyq5rqwzqfqypqhp58yjmdan79s6qqdhdzgynm4zwqd5d7xmw5fk98klysy043l2ahrqs9qrsgq7ea976txfraylvgzuxs8kgcw23ezlrszfnh8r6qtfpr6cxga50aj6txm9rxrydzd06dfeawfk6swupvz4erwnyutnjq7x39ymw6j38gp7ynn44
```

Ожидается: сумма `20m` → `0.02 BTC` = `2000000 sats` = `2000000000 msats`,
`d=—`, `description_hash=3925b6f67e2c340036ed12093dd44e0368df1b6ea26c53dbe4811f58fd5db8c1`
(SHA256 строки про chocolate cake из спеки), `expiry=3600` по умолч.
Статус: PASS.

## T4. lntb20m fallback (testnet, P2PKH)

Инвойс: кнопка 4 (`lntb20m1pvjluez…ghe9k8`).
Ожидается: сеть `testnet`, сумма `20m`, `h` как в T3, `f=1`,
`p`= тот же hash. Статус: PASS.

## T5. lnbc20m route hints (r)

Инвойс: кнопка 5 (`lnbc20m1pvjluez…76cqw0`).
Ожидается: `r-полей=1`, хопов `2` (два pubkey `029e…` / `039e…` из Breakdown),
`f=1` (P2PKH `1RustyRX…`). Статус: PASS.

## Негативные кейсы (все должны дать красную ОШИБКУ, не OK)

- N1 мусор `hello world` → ошибка bech32/префикса.
- N2 пустой ввод → «пустой ввод».
- N3 смешанный регистр (`lnbc25M1…`) → «смешанный регистр».
- N4 без `1` (`pvjluezpp5…`) → «нет разделителя».
- N5 битая контрольная сумма (последние 6 символов T2 заменены) → «контрольная сумма».
- N6 неверный префикс (`lnzz…`) → «префикс не lnbc/lntb…».
- N7 обрезанный T1 (отрезать хвост подписи) → «слишком короткая/обрезана».

## Результат прогона (round 197)

- `http 200`: проверено `curl` (см. отчёт воркера).
- T1–T5: PASS в браузере (кнопки 1–5) + node-реплика bech32-декода.
- N1–N7: честные красные ошибки, без ложного OK.
