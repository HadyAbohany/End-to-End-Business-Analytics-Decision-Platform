# Dataset Audit

Audit date: 2026-11-16. Full scans of all nine CSVs, using existing Python/pandas in memory. No data files changed or derived datasets saved.

## Source and Method

`data/raw/` does not exist in this workspace. Actual inspected source: `/Users/hadyahmed/Downloads/archive-3/`. Local version/download date and byte-for-byte identity with Kaggle are unknown. SHA-256 below identifies this exact local snapshot. Blank CSV fields are counted as null using pandas defaults; IDs and ZIP prefixes are strings. Numeric types below are inferred, not source constraints. Duplicate counts mean excess rows beyond the first. Timestamp comparisons use source naive wall time; timezone is undocumented.

## olist_customers_dataset.csv

Rows: **99,441**; bytes: 9,033,957; full-row duplicates: **0**; candidate key `customer_id` excess duplicates: **0**.

SHA-256: `983a422239e1712ded753b3bf9ecf47dc73f144d306029dcfa99e70a226883d2`

| Exact column | Parsed type | Null rows | Distinct non-null | Range (numeric or timestamp) |
|---|---|---:|---:|---|
| customer_id | string | 0 | 99,441 | — |
| customer_unique_id | string | 0 | 96,096 | — |
| customer_zip_code_prefix | string | 0 | 14,994 | — |
| customer_city | str | 0 | 4,119 | — |
| customer_state | str | 0 | 27 | — |

## olist_geolocation_dataset.csv

Rows: **1,000,163**; bytes: 61,273,883; full-row duplicates: **261,831**; candidate key `geolocation_zip_code_prefix` excess duplicates: **981,148**.

SHA-256: `b514f6fc991b9566aeba02aa5d67e2c3630f034b60a0e05aa0d082a3b66d88d6`

| Exact column | Parsed type | Null rows | Distinct non-null | Range (numeric or timestamp) |
|---|---|---:|---:|---|
| geolocation_zip_code_prefix | string | 0 | 19,015 | — |
| geolocation_lat | float64 | 0 | 717,363 | -36.6053744107061 → 45.06593318269697 |
| geolocation_lng | float64 | 0 | 717,615 | -101.46676644931476 → 121.10539381057764 |
| geolocation_city | str | 0 | 8,011 | — |
| geolocation_state | str | 0 | 27 | — |

## olist_order_items_dataset.csv

Rows: **112,650**; bytes: 15,438,671; full-row duplicates: **0**; candidate key `order_id, order_item_id` excess duplicates: **0**.

SHA-256: `0bc4d068c4fe38cbb01bd90e8746e3c613fe7b4baef75fab7b0e329701c3e279`

| Exact column | Parsed type | Null rows | Distinct non-null | Range (numeric or timestamp) |
|---|---|---:|---:|---|
| order_id | string | 0 | 98,666 | — |
| order_item_id | string | 0 | 21 | — |
| product_id | string | 0 | 32,951 | — |
| seller_id | string | 0 | 3,095 | — |
| shipping_limit_date | str | 0 | 93,318 | 2016-09-19 00:15:34 → 2020-04-09 22:35:08; invalid nonblank=0 |
| price | float64 | 0 | 5,968 | 0.85 → 6735.0 |
| freight_value | float64 | 0 | 6,999 | 0.0 → 409.68 |

## olist_order_payments_dataset.csv

Rows: **103,886**; bytes: 5,777,138; full-row duplicates: **0**; candidate key `order_id, payment_sequential` excess duplicates: **0**.

SHA-256: `4f713964f2815dbbaa40b9488268c55aac3627bfce5aa96cf58d1f3616de3cc0`

| Exact column | Parsed type | Null rows | Distinct non-null | Range (numeric or timestamp) |
|---|---|---:|---:|---|
| order_id | string | 0 | 99,440 | — |
| payment_sequential | int64 | 0 | 29 | 1 → 29 |
| payment_type | str | 0 | 5 | — |
| payment_installments | int64 | 0 | 24 | 0 → 24 |
| payment_value | float64 | 0 | 29,077 | 0.0 → 13664.08 |

## olist_order_reviews_dataset.csv

Rows: **99,224**; bytes: 14,451,670; full-row duplicates: **0**; candidate key `review_id, order_id` excess duplicates: **0**.

SHA-256: `012b61c7593e34f51fa614efdf802b9c7056ce6aae5307ddb93236e7cfc797d7`

| Exact column | Parsed type | Null rows | Distinct non-null | Range (numeric or timestamp) |
|---|---|---:|---:|---|
| review_id | string | 0 | 98,410 | — |
| order_id | string | 0 | 98,673 | — |
| review_score | int64 | 0 | 5 | 1 → 5 |
| review_comment_title | str | 87,656 | 4,527 | — |
| review_comment_message | str | 58,247 | 36,159 | — |
| review_creation_date | str | 0 | 636 | 2016-10-02 00:00:00 → 2018-08-31 00:00:00; invalid nonblank=0 |
| review_answer_timestamp | str | 0 | 98,248 | 2016-10-07 18:32:28 → 2018-10-29 12:27:35; invalid nonblank=0 |

## olist_orders_dataset.csv

Rows: **99,441**; bytes: 17,654,914; full-row duplicates: **0**; candidate key `order_id` excess duplicates: **0**.

SHA-256: `8df58ef3d2d7e9944010f7beecd9b75367f5588ec6e3c91cec19ae3345ef9ecf`

| Exact column | Parsed type | Null rows | Distinct non-null | Range (numeric or timestamp) |
|---|---|---:|---:|---|
| order_id | string | 0 | 99,441 | — |
| customer_id | string | 0 | 99,441 | — |
| order_status | str | 0 | 8 | — |
| order_purchase_timestamp | str | 0 | 98,875 | 2016-09-04 21:15:19 → 2018-10-17 17:30:18; invalid nonblank=0 |
| order_approved_at | str | 160 | 90,733 | 2016-09-15 12:16:38 → 2018-09-03 17:40:06; invalid nonblank=0 |
| order_delivered_carrier_date | str | 1,783 | 81,018 | 2016-10-08 10:34:01 → 2018-09-11 19:48:28; invalid nonblank=0 |
| order_delivered_customer_date | str | 2,965 | 95,664 | 2016-10-11 13:46:32 → 2018-10-17 13:22:46; invalid nonblank=0 |
| order_estimated_delivery_date | str | 0 | 459 | 2016-09-30 00:00:00 → 2018-11-12 00:00:00; invalid nonblank=0 |

## olist_products_dataset.csv

Rows: **32,951**; bytes: 2,379,446; full-row duplicates: **0**; candidate key `product_id` excess duplicates: **0**.

SHA-256: `3e6569628a17fbc75fd206ee357b59e20364b9afa90f5b6cd5b4d624c58aa9cc`

| Exact column | Parsed type | Null rows | Distinct non-null | Range (numeric or timestamp) |
|---|---|---:|---:|---|
| product_id | string | 0 | 32,951 | — |
| product_category_name | str | 610 | 73 | — |
| product_name_lenght | float64 | 610 | 66 | 5.0 → 76.0 |
| product_description_lenght | float64 | 610 | 2,960 | 4.0 → 3992.0 |
| product_photos_qty | float64 | 610 | 19 | 1.0 → 20.0 |
| product_weight_g | float64 | 2 | 2,204 | 0.0 → 40425.0 |
| product_length_cm | float64 | 2 | 99 | 7.0 → 105.0 |
| product_height_cm | float64 | 2 | 102 | 2.0 → 105.0 |
| product_width_cm | float64 | 2 | 95 | 6.0 → 118.0 |

## olist_sellers_dataset.csv

Rows: **3,095**; bytes: 174,703; full-row duplicates: **0**; candidate key `seller_id` excess duplicates: **0**.

SHA-256: `1f643d2b950373b85735e7794b20986f528d7a000432e7c6f9bcbb44d0846a0e`

| Exact column | Parsed type | Null rows | Distinct non-null | Range (numeric or timestamp) |
|---|---|---:|---:|---|
| seller_id | string | 0 | 3,095 | — |
| seller_zip_code_prefix | string | 0 | 2,246 | — |
| seller_city | str | 0 | 611 | — |
| seller_state | str | 0 | 23 | — |

## product_category_name_translation.csv

Rows: **71**; bytes: 2,613; full-row duplicates: **0**; candidate key `product_category_name` excess duplicates: **0**.

SHA-256: `a81f0d1f27b27e7293f761bc79e3ce8f348ee39c4b3ed3e49bde38f478586278`

| Exact column | Parsed type | Null rows | Distinct non-null | Range (numeric or timestamp) |
|---|---|---:|---:|---|
| product_category_name | str | 0 | 71 | — |
| product_category_name_english | str | 0 | 71 | — |

## Foreign Key Coverage

```text
orders.customer_id -> customers.customer_id: null=0, nonnull unmatched rows=0 / 99441
order_items.order_id -> orders.order_id: null=0, nonnull unmatched rows=0 / 112650
order_items.product_id -> products.product_id: null=0, nonnull unmatched rows=0 / 112650
order_items.seller_id -> sellers.seller_id: null=0, nonnull unmatched rows=0 / 112650
order_payments.order_id -> orders.order_id: null=0, nonnull unmatched rows=0 / 103886
order_reviews.order_id -> orders.order_id: null=0, nonnull unmatched rows=0 / 99224
products.product_category_name -> translation.product_category_name: null=610, nonnull unmatched rows=13 / 32341
customers.customer_zip_code_prefix -> geolocation.geolocation_zip_code_prefix: null=0, nonnull unmatched rows=278 / 99441
sellers.seller_zip_code_prefix -> geolocation.geolocation_zip_code_prefix: null=0, nonnull unmatched rows=7 / 3095
```

## Status and Child Multiplicity

```text
order_status
delivered      96478
shipped         1107
canceled         625
unavailable      609
invoiced         314
processing       301
created            5
approved           2
items: orders without rows=775, max rows/order=21, multirow orders=9803
payments: orders without rows=1, max rows/order=29, multirow orders=2961
reviews: orders without rows=768, max rows/order=3, multirow orders=547
multiple sellers per order=1278
review_id excess duplicates=814
```

## Status Versus Timestamp Missingness

```text
              orders  missing_approval  missing_carrier  missing_delivery
order_status
approved           2                 0                2                 2
canceled         625               141              550               619
created            5                 5                5                 5
delivered      96478                14                2                 8
invoiced         314                 0              314               314
processing       301                 0              301               301
shipped         1107                 0                0              1107
unavailable      609                 0              609               609
```

## Chronology and Delivery Label Audit

```text
{'order_approved_at before order_purchase_timestamp': 0, 'order_delivered_carrier_date before order_purchase_timestamp': 166, 'order_delivered_carrier_date before order_approved_at': 1359, 'order_delivered_customer_date before order_purchase_timestamp': 0, 'order_delivered_customer_date before order_delivered_carrier_date': 23, 'order_estimated_delivery_date before order_purchase_timestamp': 0, 'delivery label eligible all statuses': 96476, 'late exact timestamp': 7827, 'late calendar date': 6535, 'delivered status without delivery': 8, 'non-delivered status with delivery': 6}
```

## Review Label Audit

```text
{'score_distribution_rows': {1: 11424, 2: 3151, 3: 8179, 4: 19142, 5: 57328}, 'negative_rows': 14575, 'first_review_orders': 98673, 'first_review_negative': 14476, 'answer_before_creation': 0, 'answer_before_purchase': 63, 'first_answer_at_or_before_delivery': 4769, 'first_answer_after_delivery': 91061, 'reviewed_without_delivery': 2843}
```

## Delivery-time Satisfaction Cohort

```text
{'eligible': 91061, 'negative': 8612, 'answer_delay_days_quantiles': {0.5: 2.015474537037037, 0.9: 5.446504629629629, 0.95: 7.9586805555555555, 0.99: 24.1200671296295, 1.0: 518.9790740740741}}
```

## Purchase Time Coverage

```text
order_purchase_timestamp
2016-09       4
2016-10     324
2016-12       1
2017-01     800
2017-02    1780
2017-03    2682
2017-04    2404
2017-05    3700
2017-06    3245
2017-07    4026
2017-08    4331
2017-09    4285
2017-10    4631
2017-11    7544
2017-12    5673
2018-01    7269
2018-02    6728
2018-03    7211
2018-04    6939
2018-05    6873
2018-06    6167
2018-07    6292
2018-08    6512
2018-09      16
2018-10       4
Freq: M
```

## Weekly Density

```text
{'all_calendar_weeks': 112, 'zero_weeks': 11, 'stable_full_weeks': 86, 'stable_min_median_max': [46, 1005.0, 3008]}
```

## Repeat Customers

```text
{'unique_buyers': 96096, 'buyers_with_multiple_orders': 2997, 'maximum_orders_per_buyer': 17}
```

## Category Support

```text
                                                items  orders
product_category_name
seguros_e_servicos                                  2       2
fashion_roupa_infanto_juvenil                       8       8
pc_gamer                                            9       8
cds_dvds_musicais                                  14      12
la_cuisine                                         14      13
portateis_cozinha_e_preparadores_de_alimentos      15      14
artes_e_artesanato                                 24      23
casa_conforto_2                                    30      24
fraldas_higiene                                    39      27
fashion_esporte                                    30      27
flores                                             33      29
moveis_colchao_e_estofado                          38      38
musica                                             38      38
fashion_roupa_feminina                             48      39
artigos_de_festas                                  43      39
livros_importados                                  60      53
dvds_blu_ray                                       64      59
cine_foto                                          72      65
portateis_casa_forno_e_cafe                        76      75
tablets_impressao_imagem                           83      79
moveis_quarto                                     109      95
construcao_ferramentas_ferramentas                103      97
fashion_roupa_masculina                           132     112
fashion_underwear_e_moda_praia                    131     121
artigos_de_natal                                  153     128
sinalizacao_e_seguranca                           199     140
construcao_ferramentas_seguranca                  194     167
pcs                                               203     181
agro_industria_e_comercio                         212     182
construcao_ferramentas_jardim                     238     194
artes                                             209     202
telefonia_fixa                                    264     217
alimentos_bebidas                                 278     227
eletrodomesticos_2                                238     234
industria_comercio_e_negocios                     268     235
fashion_calcados                                  262     240
construcao_ferramentas_iluminacao                 304     244
moveis_cozinha_area_de_servico_jantar_e_jardim    281     248
climatizacao                                      297     253
livros_tecnicos                                   267     260
market_place                                      311     280
bebidas                                           379     297
audio                                             364     350
casa_conforto                                     434     397
moveis_sala                                       503     422
alimentos                                         510     450
casa_construcao                                   604     490
livros_interesse_geral                            553     512
instrumentos_musicais                             680     628
eletroportateis                                   679     630
construcao_ferramentas_construcao                 929     748
eletrodomesticos                                  771     764
malas_acessorios                                 1092    1034
consoles_games                                   1137    1062
moveis_escritorio                                1691    1273
pet_shop                                         1947    1710
fashion_bolsas_e_acessorios                      2031    1864
papelaria                                        2517    2311
eletronicos                                      2767    2550
bebes                                            3065    2885
perfumaria                                       3419    3162
ferramentas_jardim                               4347    3518
cool_stuff                                       3796    3632
brinquedos                                       4117    3886
automotivo                                       4235    3897
telefonia                                        4545    4199
relogios_presentes                               5991    5624
utilidades_domesticas                            6964    5884
moveis_decoracao                                 8334    6449
informatica_acessorios                           7827    6689
esporte_lazer                                    8641    7720
beleza_saude                                     9670    8836
cama_mesa_banho                                 11115    9417
```

## Geolocation Ambiguity

```text
{'zip_prefixes': 19015, 'prefix_multiple_states': 8, 'prefix_multiple_city_strings': 8556, 'invalid_world_coordinates': 0, 'outside_broad_brazil_box': 31}
```

## Financial Reconciliation

```text
{'item_price_sum': 13591643.7, 'freight_sum': 2251909.54, 'payment_sum': 16008872.12, 'matched_orders': 98665, 'absolute_difference_over_one_cent': 303, 'max_abs_difference': 182.81, 'sum_difference_matched': 2870.39, 'zero_payment_rows': 9, 'zero_installment_rows': 2, 'payment_types': {'credit_card': 76795, 'boleto': 19784, 'voucher': 5775, 'debit_card': 1529, 'not_defined': 3}}
```

## Numeric Quality

```text
items.price: negative=0, zero=0
items.freight_value: negative=0, zero=383
products.product_weight_g: negative=0, zero=4
products.product_length_cm: negative=0, zero=0
products.product_height_cm: negative=0, zero=0
products.product_width_cm: negative=0, zero=0
payments.payment_value: negative=0, zero=9
payments.payment_installments: negative=0, zero=2
```

## Label Maturity at Candidate Cutoffs

```text
{'cutoff_exclusive': '2018-01-01', 'purchases_before': 45430, 'delivery_labels_before': 41202, 'delivery_late_day_before': 1765, 'delivery_time_review_labels_before': 38997, 'negative_delivery_time_reviews_before': 3603}
{'cutoff_exclusive': '2018-04-01', 'purchases_before': 66638, 'delivery_labels_before': 60474, 'delivery_late_day_before': 3608, 'delivery_time_review_labels_before': 56726, 'negative_delivery_time_reviews_before': 5499}
{'cutoff_exclusive': '2018-06-01', 'purchases_before': 80450, 'delivery_labels_before': 75435, 'delivery_late_day_before': 5369, 'delivery_time_review_labels_before': 70492, 'negative_delivery_time_reviews_before': 6841}
{'cutoff_exclusive': '2018-09-01', 'purchases_before': 99421, 'delivery_labels_before': 96417, 'delivery_late_day_before': 6476, 'delivery_time_review_labels_before': 90414, 'negative_delivery_time_reviews_before': 8562}
{'cutoff_exclusive': '2018-10-18', 'purchases_before': 99441, 'delivery_labels_before': 96476, 'delivery_late_day_before': 6535, 'delivery_time_review_labels_before': 91052, 'negative_delivery_time_reviews_before': 8611}
```

## Table Grains and Interpretation

Customers: one order-specific customer record, not one person. Orders: one order. Items: one numbered line per order. Payments: one payment sequence per order. Reviews: one review/order association, not one unique review ID globally. Products and sellers: one entity snapshot. Geolocation: repeated location observations, not a ZIP dimension. Translation: one Portuguese category mapping. No inferred primary key is imposed on raw geolocation.

## Assumptions / Evidence / Open Questions

Evidence: complete scans above; counts are snapshot facts, not evidence of predictive performance. Future feasibility uses this audit, not approximate counts in the brief. Timestamp semantics and original extract completeness remain uncertain. Review answer is the conservative label-release proxy; survey creation is not answer availability. No cancellation timestamp exists. Static catalog, address, promise and item availability at placement must be explicitly assumed.

Provenance: [official Kaggle listing](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) was opened but provided no readable page content. Therefore publisher/version/license binding to these exact bytes is unverified. The supplied brief identifies Olist and CC BY-NC-SA 4.0. The [Creative Commons deed](https://creativecommons.org/licenses/by-nc-sa/4.0/) was readable and confirms attribution, noncommercial and share-alike conditions for material under that license; this does not establish that these local files carry it. Before external redistribution, retain original source/version/license evidence. No raw data is redistributed by this plan.

See [feasibility](02_FEASIBILITY_MATRIX.md), [model design](04_ML_MODEL_DESIGN.md) and [replay](05_REPLAY_AND_DATA_DESIGN.md).

## Candidate Window Support

These are retrospectively observed cohort counts, not necessarily labels available at the window end. M1 cohorts use purchase time; M2 cohorts use delivery time.

2017-01-01 to 2018-01-01 exclusive: purchases=45101, observed deliveries=43426, calendar-late=2453, delivery-time reviewed cohort=39351, negatives=3619

2018-01-01 to 2018-04-01 exclusive: purchases=21208, observed deliveries=20628, calendar-late=2658, delivery-time reviewed cohort=17733, negatives=1917

2018-04-01 to 2018-06-01 exclusive: purchases=13812, observed deliveries=13547, calendar-late=749, delivery-time reviewed cohort=13474, negatives=1294

2018-06-01 to 2018-08-27 exclusive: purchases=18841, observed deliveries=18487, calendar-late=672, delivery-time reviewed cohort=18691, negatives=1621

## Additional Quality Checks

Shipping deadlines before purchase: 0 item rows; after 2018-12-31: 4 item rows. Deadlines are not event times.

Unmapped nonnull product categories: pc_gamer, portateis_cozinha_e_preparadores_de_alimentos

After removing pre-purchase review answers and selecting the first remaining answer per order: 98,613 reviewed orders; 91,064 delivery-time labeled cases; 8,613 negatives. This supplements, rather than silently replaces, the raw first-answer diagnostic.

Source status quality does not justify a delivered-only feature-time filter. Model labels use valid observed timestamps independently of final status.

## Final Planning Verification

All nine source SHA-256 hashes were recalculated after document creation and matched the initial inventory. Eleven required Markdown documents exist; local cross-links resolve, Markdown code fences are balanced, and eleven Mermaid source diagrams are present. No rendered Mermaid validation, application test, model fitting or infrastructure execution was performed. Repository status shows only the new planning directory.
