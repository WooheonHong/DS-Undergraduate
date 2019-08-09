EDA
================
true
2018-8 ~ 2018-9

``` r
file_name <- dir(pattern = "^(train)(.)*\\.fst$")
games <- file_name %>% 
  map(fst::read_fst) %>% 
  map(as.tibble)
names(games) <- str_remove(file_name, ".fst")
map2(names(games), games, assign, envir = globalenv())
rm(games)
```

# 파티 모델링 데이터

## 파티데이터 전처리

1/1000초는 제거하였다. 파티 지속 시간 변수를 추가하였다. 파티원 수 변수 생성

``` r
train_party <- train_party %>% 
  separate(party_start_time, c("party_start_time", "sec_start"), "\\.") %>% 
  separate(party_end_time, c("party_end_time", "sec_end"), "\\.") %>% 
  select(-c(sec_start, sec_end))

train_party <- train_party %>% 
  mutate(
    start_day =  ymd(20180331) + (party_start_week - 1)* 7 + party_start_day,
    end_day = ymd(20180331) + (party_end_week - 1)* 7 + party_end_day,
    start_daytime = ymd_hms(paste(start_day, party_start_time, sep = " ")),
    end_daytime = ymd_hms(paste(end_day, party_end_time, sep = " ")),
    diff_minute = (start_daytime %--% end_daytime) / minutes(1),
    cnt_party_member = str_count(hashed, "\\,") + 1
  )
fst::write_fst(train_party, "train_party.fst")
```

``` r
party_time <- function(binwidth = 1, xlim = c(NA_real_, NA_real_)) {
  ggplot(train_party, aes(diff_minute)) +
    geom_histogram(binwidth = binwidth) +
    xlim(xlim)
}

gridExtra::grid.arrange(
  party_time(binwidth = 1/6, xlim = c(NA, 60)) + ggtitle("10초 단위"),
  party_time(binwidth = 0.5, xlim = c(NA, 60))+ ggtitle("30초 단위"),
  party_time(xlim = c(NA, 60))+ ggtitle("1분 단위"),
  party_time(binwidth = 2, xlim = c(NA, 60))+ ggtitle("2분 단위"),
  ncol = 2)

gridExtra::grid.arrange(
party_time(xlim = c(60, 1440)) + ggtitle("1시간 ~ 24시간, 1분단위"),
party_time(binwidth = 10, xlim = c(1440, NA_real_)) + ggtitle("24시간이상, 10분단위"),
ncol = 2)
```

## 파티 모델링 데이터 생성

``` r
temp <- train_party[, c("party_start_week", "hashed", "party_start_day")] %>%
   mutate(id = row_number(),
     party_start_week = as.factor(str_c("wk_", party_start_week))) %>%
  select(id, everything()) %>% 
  spread(party_start_week, hashed)

df <- vector("list", 8)
for(i in 1:8) {
  df[[i]] <- temp %$% 
  str_split(get(str_c("wk_", i)), "\\,") %>% 
  flatten_chr() %>% 
  tibble(acc_party_id = .) %>%
  count(acc_party_id)
}
df[[1]] <- rename(df[[1]], cnt_party_wk1 = n)
df[[2]] <- rename(df[[2]], cnt_party_wk2 = n)
df[[3]] <- rename(df[[3]], cnt_party_wk3 = n)
df[[4]] <- rename(df[[4]], cnt_party_wk4 = n)
df[[5]] <- rename(df[[5]], cnt_party_wk5 = n)
df[[6]] <- rename(df[[6]], cnt_party_wk6 = n)
df[[7]] <- rename(df[[7]], cnt_party_wk7 = n)
df[[8]] <- rename(df[[8]], cnt_party_wk8 = n)
df <- df %>% reduce(full_join)

model_party <- df; rm(df)
model_party <- model_party %>% 
  semi_join(train_activity, by = c("acc_party_id" = "acc_id")) %>% 
  rename(acc_id = acc_party_id)

temp <- train_activity %>% 
  anti_join(model_party) %>% 
  mutate(cnt_party = 0) %>% 
  select(acc_id, wk, cnt_party) %>% 
  complete(acc_id, wk) %>% 
  spread(wk, cnt_party) %>% 
  rename(cnt_party_wk1 = `1`,
         cnt_party_wk2 = `2`,
         cnt_party_wk3 = `3`,
         cnt_party_wk4 = `4`,
         cnt_party_wk5 = `5`,
         cnt_party_wk6 = `6`,
         cnt_party_wk7 = `7`,
         cnt_party_wk8 = `8`) 

model_party <- bind_rows(model_party, temp) %>% 
  gather(wk, cnt_party, -acc_id) %>% 
  mutate(cnt_party = ifelse(is.na(cnt_party), 0, cnt_party)) %>% 
  spread(wk, cnt_party)
model_party[, 2:9] <- lapply(model_party[,2:9], as.integer)
rm(temp)

write_csv(model_party, "model_train_party.csv")
fst::write_fst(model_party, "model_train_party.fst")
# model_party <- fst::read_fst("model_train_party.fst") %>% as.tibble()
```

``` r
temp <- train_party[, c("party_start_week", "hashed", "party_start_day")] %>%
   mutate(id = row_number(),
     party_start_week = as.factor(str_c("wk_", party_start_week))) %>%
  select(id, everything()) %>% 
  spread(party_start_week, hashed)

df <- vector("list", 8)
for(i in 1:8) {
  df[[i]] <- temp %$% 
  str_split(get(str_c("wk_", i)), "\\,") %>% 
  flatten_chr() %>% 
  tibble(acc_party_id = .) %>%
  count(acc_party_id)
}
df[[1]] <- rename(df[[1]], cnt_party_wk1 = n)
df[[2]] <- rename(df[[2]], cnt_party_wk2 = n)
df[[3]] <- rename(df[[3]], cnt_party_wk3 = n)
df[[4]] <- rename(df[[4]], cnt_party_wk4 = n)
df[[5]] <- rename(df[[5]], cnt_party_wk5 = n)
df[[6]] <- rename(df[[6]], cnt_party_wk6 = n)
df[[7]] <- rename(df[[7]], cnt_party_wk7 = n)
df[[8]] <- rename(df[[8]], cnt_party_wk8 = n)
df <- df %>% reduce(full_join)

model_party <- df; rm(df)
model_party <- model_party %>% 
  semi_join(train_activity, by = c("acc_party_id" = "acc_id")) %>% 
  rename(acc_id = acc_party_id)

temp <- train_activity %>% 
  anti_join(model_party) %>% 
  mutate(cnt_party = 0) %>% 
  select(acc_id, wk, cnt_party) %>% 
  complete(acc_id, wk) %>% 
  spread(wk, cnt_party) %>% 
  rename(cnt_party_wk1 = `1`,
         cnt_party_wk2 = `2`,
         cnt_party_wk3 = `3`,
         cnt_party_wk4 = `4`,
         cnt_party_wk5 = `5`,
         cnt_party_wk6 = `6`,
         cnt_party_wk7 = `7`,
         cnt_party_wk8 = `8`) 

model_party <- bind_rows(model_party, temp) %>% 
  gather(wk, cnt_party, -acc_id) %>% 
  mutate(cnt_party = ifelse(is.na(cnt_party), 0, cnt_party)) %>% 
  spread(wk, cnt_party)
model_party[, 2:9] <- lapply(model_party[,2:9], as.integer)
rm(temp)
```

요일별 파티 참여 수

1이 수요일이고 7이 화요일이다.

``` r
train_party %$% 
  table(party_start_day)
```

``` r
party_week <- train_party %>% 
  select(party_start_week, party_start_day, hashed) %>% 
  group_by(party_start_week, party_start_day) %>% 
  mutate(acc_id = str_split(hashed, "\\,")) %>% 
  select(-hashed) %>% 
  unnest(acc_id) %>% 
  semi_join(train_activity) %>% 
  group_by(acc_id, party_start_day) %>% 
  summarise(n = n()) %>% 
  complete(acc_id, party_start_day, fill = list(n = 0)) %>% 
  ungroup()

write_csv(party_week, "party_train_week.csv")

df1 <- party_week %>% 
  group_by(acc_id) %>% 
  summarise(sd_party_week = sd(n))
df2 <- party_week %>% 
  filter(party_start_day == 1) %>% 
  group_by(acc_id) %>% 
  summarise(cnt_party_wednes = n)
df3 <- party_week %>% 
  filter(party_start_day %in% c(4,5)) %>% 
  group_by(acc_id) %>% 
  summarise(cnt_party_weekend = sum(n))
df <- full_join(df1, df2) %>% full_join(df3)
df <- df %>% 
  mutate(sd_party_week = ifelse(is.nan(sd_party_week), 0, sd_party_week),
         cnt_party_wednes = ifelse(is.na(cnt_party_wednes), 0, cnt_party_wednes),
         cnt_party_weekend = ifelse(is.na(cnt_party_weekend), 0, cnt_party_weekend))

temp <- train_label %>% 
  anti_join(df) %>% 
  mutate(sd_party_week = 0,
         cnt_party_wednes = 0,
         cnt_party_weekend = 0) %>% 
  select(-label)

df <- bind_rows(df, temp)
```

파티 지속시간 합, 8주 파티의 지속시간 합

``` r
party_duration <- train_party %>% 
  select(party_start_week, diff_minute, hashed) %>% 
  group_by(party_start_week) %>% 
  mutate(acc_id = str_split(hashed, "\\,")) %>% 
  select(-hashed) %>% 
  unnest(acc_id) %>% 
  filter(acc_id %in% train_label$acc_id) %>% 
  group_by(acc_id, party_start_week) %>% 
  summarise(duration_party = sum(diff_minute)) %>% 
  complete(acc_id, party_start_week, fill = list(duration_party = 0)) %>% 
  ungroup()

party_duration_sum <- party_duration %>% 
  group_by(acc_id) %>% 
  summarise(sum_duration_party = sum(duration_party)) 

party_duration_wk8 <- party_duration %>%
  filter(party_start_week == 8) %>%
  select(acc_id, duration_party) %>% 
  full_join(party_duration_sum) %>% 
  rename(duration_party = wk8_duration_party) %>% 
  mutate(wk8_duration_party = ifelse(is.na(wk8_duration_party), 0, wk8_duration_party))

party_duration <- party_duration_wk8 %>% 
  mutate(wk8_duration_party = ifelse(is.na(wk8_duration_party), 0, wk8_duration_party))  

temp <- train_label %>% 
  anti_join(party_duration) %>% 
  mutate(sum_duration_party = 0,
         wk8_duration_party = 0)
party_duration <- bind_rows(party_duration_wk8, temp) %>% 
  select(-label)
rm(temp)

write_csv(party_duration, "party_train_duration.csv")
```

``` r
model_train_party <- left_join(model_train_party, party_duration) %>% 
  left_join(df)
write_csv(model_train_party, "model_train_party.csv")
```

# 길드 모델링 데이터

``` r
guild_activity_id <- train_guild %>% 
  group_by(guild_id) %>% 
  mutate(acc_id = str_split(guild_member_acc_id, "\\,")) %>% 
  select(-guild_member_acc_id) %>% 
  unnest(acc_id) %>% 
  arrange(acc_id) %>% 
  group_by(acc_id) %>% 
  summarise(cnt_guild = max(cnt_guild)) %>% 
  filter(acc_id %in% unique(train_activity$acc_id))

not_guild <- train_activity %>% 
  select(acc_id) %>% 
  distinct(acc_id) %>% 
  anti_join(guild_activity_id) %>% 
  mutate(cnt_guild = 0)

model_guild <- bind_rows(guild_activity_id, not_guild)
rm(list = c("guild_activity_id", "not_guild"))
# write_csv(model_guild, "model_train_guild.csv")
# fst::write_fst(model_guild, "model_train_guild.fst")
```

# 과금 모델링 데이터

``` r
payment8 <- train_payment %>% 
  filter(payment_week == 8, payment_amount) %>% 
  rename(payment_amount8 = payment_amount) %>% 
  select(-payment_week)

model_payment <- train_payment %>% 
  group_by(acc_id) %>% 
  summarise(sum_payment = sum(payment_amount, na.rm = TRUE)) %>% 
  mutate(
    no_payment = ifelse(sum_payment == min(sum_payment), TRUE, FALSE)) %>% 
  left_join(payment8)
rm(payment8)
#write_csv(model_payment, "model_train_payment.csv")
#fst::write_fst(model_payment, "model_train_payment.fst")
```

# 거래 모델링 데이터

``` r
train_trade <- train_trade %>% 
  distinct() 
      
cnt_source <- train_trade %>%
  group_by(source_acc_id) %>%
  summarise(cnt_source = n(),
           sum_source_item = sum(item_amount))
  
cnt_target <- train_trade %>% 
  group_by(target_acc_id) %>% 
  summarise(cnt_target = n(),
            sum_target_item = sum(item_amount)) 

# NA = 거래 횟수 0이므로 대체한다.
model_trade <- distinct(train_activity[, "acc_id"]) %>% 
  left_join(cnt_source, by = c("acc_id" = "source_acc_id")) %>% 
  left_join(cnt_target, by = c("acc_id" = "target_acc_id")) %>% 
  mutate(cnt_source = ifelse(is.na(cnt_source), 0, cnt_source),
         cnt_target = ifelse(is.na(cnt_target), 0, cnt_target),
         sum_source_item = ifelse(is.na(sum_source_item), 0, sum_source_item),
         sum_target_item = ifelse(is.na(sum_target_item), 0, sum_target_item))

# write_csv(model_trade, "model_train_trade.csv")
# fst::write_fst(model_trade, "model_train_trade.fst")
```

# 활동 모델링 데이터

## 카운트 변환

``` r
reToCNT <- function(vec){
  # This is the function that return count value from standardized vector
  sort_unique <- sort(unique(vec))
  diff_unique <- diff(sort_unique)
  min_diff <- min(diff_unique)
  min_zero <- vec - min(vec) 
  
  return(min_zero/min_diff)
}

train_activity[, 12:38] <- lapply(train_activity[, 12:38], reToCNT)
train_activity[, 12:38] <- lapply(train_activity[, 12:38], as.integer) 
```

## 경향성

주간 접속 횟수, 플레이 시간, 전투 시간, 재화 획득 시간에 따른 경향성과 합

cbind를 사용하지않으면 굉장히 느려진다.

``` r
model_train_activity <- vector("list", 10)
model_train_activity[[1]] <- train_activity %>%
  group_by(acc_id) %>%
  summarise(sum_playtime = sum(play_time),
            sum_combattime = sum(game_combat_time),
            sum_money = sum(get_money),
            sum_dt = sum(cnt_dt),
            cnt_access = n())

# trend
trend_fun <- function(df) {
  lm(cbind(play_time, game_combat_time, cnt_dt, npc_exp, quest_exp, quest_hongmun, item_hongmun, cnt_use_buffitem, sum_chat2) ~ wk, data = df)$coefficients[2, ]
}

model_train_activity[[2]] <- train_activity %>%
   mutate(sum_chat2 = normal_chat + whisper_chat + district_chat + party_chat + guild_chat + faction_chat) %>% 
  nest(-acc_id) %>%
  mutate(
    trend = map(data, trend_fun)) %>% 
  unnest(trend) %>% 
  mutate(trend_type = rep(c("play_time", "game_combat_time", "cnt_dt", "npc_exp", "quest_exp", "quest_hongmun", "item_hongmun", "cnt_use_buffitem", "sum_chat2"), 100000),
         trend = ifelse(is.na(trend), 0, trend)) %>% 
  spread(trend_type, trend)
colnames(model_train_activity[[2]])[-1] <- c("trend_dt", "trend_buffitem", "trend_combattime", "trend_item_hongmun", "trend_npc", "trend_playtime", "trend_quest", "trend_quest_hongmun", "trend_chat") 
```

## 미접속 설명 변수

**최장 미활동 접속 시간**

1.  “long”형태가 작업하기 쉬우므로 형태를 변환한다.
2.  subject별로 그룹을 짓는다.  
3.  play\_time에서 NA끼리의 그룹을 얻기 위해 NA와 NA가 아닌 값의 두 가지 그룹으로 나누는데 각 그룹의 경계는
    NA이다.
4.  3번에서 얻은 그룹의 성분의 수를 `add_count()`로 새로운 열 n을 만든다.
5.  play\_time에서의 결측치에 해당하는 행들에 대해 n의 값을 비교해 최댓값을 NA\_consecutive로 만든다.

참고. add\_count: 첫 번째 인자 df. 두 번째 인자 Variables to group by. 세 번째 인자 wt.

**8주와 연속된 접속 수**

6.  grp의 최댓값이 8주의 그룹이고 이때 add\_count로 만든 n의 값이 연속된 접속 수이다.

**8주와 직전 접속주의 거리**

7.만약 grp의 최댓값이 1이고 n = 1이면 8주만 접속했으므로 거리는 0이다. 8주그룹이 n = 1이면 거리는
8주그룹(grp의 최댓값) - 1의 grp에 해당하는 n + 1이 거리이다. 두 경우 모두 아니라면 8주와 7주 모두
접속 했으므로 거리는 1이다.

**8만 접속한 사람**

위의 변수들이 그렇게 유의하지 않아서 만들었다.

``` r
model_train_activity[[3]] <- train_activity %>%
  select(acc_id, wk, play_time) %>% 
  complete(acc_id, wk) %>% 
  group_by(acc_id) %>%
  mutate(grp = cumsum(c(0, abs(diff(!is.na(play_time))) == 1))) %>%
  add_count(acc_id, grp) %>%
  summarise(
    NA_consecutive = 
      ifelse(sum(is.na(play_time)) != 0, max(n[is.na(play_time)]), 0),
    wk8_consecutive = n[which.max(grp)],# grp == max(grp)를 사용하려면 unique()와 함께 사용해야 한다.
   wk8_distance = ifelse(grp[which.max(grp)] == 1 & n[which.max(grp)] == 1 , 0, ifelse(n[which.max(grp)] == 1, n[which.max(grp) - 1] + 1, 1)),
   wk8_only = ifelse(wk8_distance == 0, TRUE, FALSE)
    )
```

## 경험치

저렙을 판별하는 데에 도움을 줄 홍문 여부를 변수로 넣었다.

``` r
# NPC, 퀘스트, 아이템 경험치
model_train_activity[[4]] <- train_activity %>% 
  group_by(acc_id) %>% 
  summarise(
    sum_npc = sum(npc_exp),
    sum_npc_hongmun = sum(npc_hongmun),
    sum_quest = sum(quest_exp),
    sum_quest_hongmun = sum(quest_hongmun),
    sum_item_hongmun = sum(item_hongmun),
    sum_hongmun = sum(sum_npc_hongmun, sum_quest_hongmun, sum_item_hongmun)) %>% 
  mutate(is_hongmun = ifelse(sum_hongmun == min(sum_hongmun), FALSE, TRUE)) %>% 
  select(-sum_hongmun)
```

홍문을 들지 않은 인원은 3675명

## 카운트변수

### 총합과 승률

``` r
# 결투, 전장
model_train_activity[[5]] <- train_activity %>% 
  group_by(acc_id) %>% 
  summarise(
    sum_duel = sum(duel_cnt),
    sum_duel_win = sum(duel_win),
    sum_partybattle = sum(partybattle_cnt),
    sum_partybattle_win = sum(partybattle_win),
    avg_duel = sum_duel_win / sum_duel,
    avg_duel = sum_partybattle_win / sum_partybattle,
  ) %>% 
  select(-contains("win"))
```

  - 바람평야 x : 97935

  - light인던 x : 57213

  - 숙련 인던 클리어 x : 90803

  - 레이드 light 참여 x : 83297

  - 레이드 light 클리어 x :

  - 레이드 참여 x : 86343

  - 레이드 클리어 x :

<!-- end list -->

``` r
# 인던, 레이드, 바람평야, 버프
change_var_name <- function(data, paste) {
  str_remove(data, "cnt_") %>% 
    str_c(paste, .)
}

model_train_activity[[6]] <- train_activity %>% 
  group_by(acc_id) %>% 
  summarise_at(vars(contains("cnt_")), sum, na.rm = TRUE) %>% 
  rename_at(vars(contains("cnt_")), ~change_var_name(., "sum_")) %>% 
  mutate(
    avg_inzone_solo = sum_clear_inzone_solo / sum_enter_inzone_solo,
    avg_inzone_light = sum_clear_inzone_light / sum_enter_inzone_light,
    avg_inzone_skilled = sum_clear_inzone_skilled / sum_enter_inzone_skilled,
    avg_inzone_normal = sum_clear_inzone_normal / sum_enter_inzone_normal,
    avg_raid = sum_clear_raid / sum_enter_raid,
    avg_raid_light = sum_clear_raid_light / sum_enter_raid_light,
    is_bam = ifelse(sum_enter_bam == 0, FALSE, TRUE),
    is_inzone_light = ifelse(sum_enter_inzone_light == 0, FALSE, TRUE),
    is_inzone_skilled = ifelse(sum_enter_inzone_skilled == 0, FALSE, TRUE),
    is_raid = ifelse(sum_enter_raid == 0, FALSE, TRUE),
    is_raid_light = ifelse(sum_enter_raid_light == 0, FALSE, TRUE),
    is_raid_clear = ifelse(sum_clear_raid == 0 , FALSE, TRUE),
    is_raid_clear_light = ifelse(sum_clear_raid_light == 0 , FALSE, TRUE)
    ) %>% 
  select(-sum_dt)
```

바람평야는 입장 수가 다른 인던에 비하여 현저히 떨어지므로 입장여부를 더미변수로 두었다.

### 채팅, 채집, 제작

``` r
# 채팅
model_train_activity[[7]] <- train_activity %>% 
  group_by(acc_id) %>% 
  summarise_at(vars(contains("chat")), sum, na.rm = TRUE) %>% 
  rename_at(vars(contains("chat")), ~str_c("sum_", .)) %>% 
  group_by(acc_id) %>% 
  mutate(sum_chat = sum(sum_normal_chat, sum_whisper_chat, sum_district_chat, sum_party_chat, sum_guild_chat, sum_faction_chat)) 

# 채집, 제작
model_train_activity[[8]] <- train_activity %>% 
  group_by(acc_id) %>% 
  summarise(sum_gathering = sum(gathering_cnt, na.rm = TRUE),
            sum_making = sum(making_cnt, na.rm = TRUE))
```

## 8주 데이터

모델링에서 중요도가 높았던 합변수들에 대해 8주 정보를 추가했다.

``` r
model_train_activity[[9]] <- train_activity %>% 
  filter(wk == 8) %>% 
  mutate(sum_chat2 = normal_chat + whisper_chat + district_chat + party_chat + guild_chat + faction_chat) %>% 
  select(acc_id, play_time, game_combat_time, npc_exp, quest_exp, quest_hongmun, item_hongmun, cnt_use_buffitem, sum_chat2)

colnames(model_train_activity[[9]])[-1] <- c("wk8_playtime", "wk8_combattime", "wk8_npc", "wk8_quest", "wk8_quest_hongmun", "wk8_itme_hongmun", "wk8_buffitem", "wk8_chat")
```

### 장기 이탈자 예측 변수

#### dt변동

``` r
model_train_activity[[10]] <- train_activity %>% 
  select(wk, acc_id, cnt_dt) %>% 
  complete(acc_id, wk, fill = list(cnt_dt = 0)) %>% 
  group_by(acc_id) %>% 
  summarise(sd_dt = sd(cnt_dt))
```

### 모델링 데이터 최종 생성

``` r
model_train_data <- model_train_activity %>%
  reduce(left_join, by = "acc_id") %>% 
  right_join(train_label, by = "acc_id") %>% 
  select(acc_id, label, everything())
#rm(model_train_activity)
```

``` r
model_train_data <- model_train_data %>% 
  left_join(model_party) %>% 
  left_join(model_guild) %>% 
  left_join(model_payment) %>% 
  left_join(model_trade)
```

NAN제거

``` r
remove_NAN <- function(data) {
  ifelse(is.nan(data), 0, data)
}
model_train_data <- model_train_data %>% 
  mutate_at(vars(contains("avg")) , remove_NAN)

write_csv(model_train_data, "model_train_data2.csv")
```

**8월 22일 추가분**

  - cnt\_dt\_1~8을 넣는다

<!-- end list -->

``` r
df <- train_activity %>% 
  select(acc_id, wk, cnt_dt) %>% 
  complete(acc_id, wk, fill = list(cnt_dt = 0)) %>% 
  spread(wk, cnt_dt) 

colnames(df)[-1] <- str_c("cnt_dt", colnames(df)[-1])

model_train_data <- left_join(model_train_data, df)
```

  - 파티 전처리 파티원 수 2명이상, 지속 시간 3분이상에서 그대로 사용
  - 파티참여수는 8주차와 총합만 사용

<!-- end list -->

``` r
model_train_data <- model_train_data %>%
  group_by(acc_id) %>% 
  mutate(sum_party = sum(cnt_party_wk1 + cnt_party_wk2 + cnt_party_wk3 + cnt_party_wk4 + cnt_party_wk5 + cnt_party_wk6 + cnt_party_wk7 + cnt_party_wk8),
         is_party = ifelse(sum_party == 0, FALSE, TRUE)) %>%
  select(-one_of( str_c("cnt_party_wk", c(2, 3, 5, 6, 7)))) %>% 
  ungroup()
```

**8월 31일 추가분**

dt 변동, 파티 요일변수, 주요 변수들의 trend, 파티 지속시간의 합, 채팅의 모든 종류 추가

접속 비율 추가, 입장 더미 추가

``` r
model_train_data <- model_train_data %>%
  mutate(
 avg_dt = sum_dt / cnt_access,
    enter = (is_inzone_light + is_inzone_skilled + is_raid + is_raid_light) / 4
  )
```

인던끼리의 합, 레이드끼리의 합, 인던 레이드의 합, npc끼리의 합, 홍문끼리의 합

``` r
model_train_data <- model_train_data %>% 
  mutate(
    sum_enter_inzone_all = sum_enter_inzone_solo + sum_enter_inzone_light + sum_enter_inzone_skilled + sum_enter_inzone_normal,
   sum_clear_inzone_all = sum_clear_inzone_solo + sum_clear_inzone_light +
   sum_clear_inzone_skilled + sum_clear_inzone_normal, 
   sum_enter_raid_all = sum_enter_raid + sum_enter_raid_light,
   sum_clear_raid_all = sum_clear_raid + sum_clear_raid_light,
   sum_enter_hunt = sum_enter_inzone_all + sum_enter_raid_all,
   sum_clear_hunt = sum_clear_inzone_all + sum_clear_raid_all,
   sum_npc_all = sum_npc + sum_npc_hongmun,
   sum_hongmun_all = sum_npc_hongmun + sum_quest_hongmun + sum_item_hongmun
  )
```

``` r
write_csv(model_train_data, "model_train_data3.csv") 
```

9월1일 추가분

play\_time의 최솟값 -0.661667373685867이 0이 아니므로 -0.661667373685867 +
0.661657483033828 만큼 -0.661667373685867에서 더한다.

quest의 -0.250423355452758의 최솟값

combattime

``` r
train_activity %>% 
  arrange(play_time) %$%
  table(play_time)

train_activity %>% 
  arrange(game_combat_time) %$%
  table(game_combat_time)

train_play_time <- train_activity %>% 
  select(acc_id, wk, play_time) %>% 
  complete(acc_id, wk, fill = list(play_time = -0.6616773)) %>%
  spread(wk, play_time) 
colnames(train_play_time)[-1] <- str_c("play_time", 1:8)

train_quest <- train_activity %>% 
  select(acc_id, wk, quest_exp) %>% 
  complete(acc_id, wk, fill = list(quest_exp = -0.2504595)) %>%
  spread(wk, quest_exp) 
colnames(train_quest)[-1] <- str_c("quest", 1:8)

train_combattime <- train_activity %>% 
  select(acc_id, wk, game_combat_time) %>% 
  complete(acc_id, wk, fill = list(game_combat_time = -0.569841814936623)) %>%
  spread(wk, game_combat_time) 
colnames(train_combattime)[-1] <- str_c("combattime", 1:8)

sd_play_time <- train_activity %>% 
  select(acc_id, wk, play_time) %>%
  complete(acc_id, wk, fill = list(play_time = -0.6616773)) %>% 
  group_by(acc_id) %>% 
  summarise(sd_play_time = sd(play_time))

sd_quest <- train_activity %>% 
  select(acc_id, wk, quest_exp) %>%
  complete(acc_id, wk, fill = list(quest_exp = -0.6616773)) %>% 
  group_by(acc_id) %>% 
  summarise(sd_quest = sd(quest_exp))

model_train_data <- model_train_data %>% 
  left_join(train_play_time) %>% 
  left_join(train_quest) %>% 
  left_join(train_combattime) %>%
  left_join(sd_play_time) %>% 
  left_join(sd_quest)

write_csv(model_train_data, "model_train_data5.csv")
```

# 레이블을 잘 설명해주는 최적변수 찾기

## 최적 변수

## 이탈/미이탈 최적 변수

``` r
model_train_data <- model_train_data %>% 
  mutate(is_raid = ifelse(sum_enter_raid %in% min(sum_enter_raid), TRUE, FALSE))
```

### 8주만 접속

``` r
model_train_data %>% 
  filter(wk8_distance == 0) %$%
  table(label)

model_train_data %>% 
  filter(wk8_distance != 0) %$%
  table(label)
```

총량이 적어서 최적은 아니지만 8주만 접속한 사람들의 이탈율은 95.3%에 달한다\! (

*레이드 하는 사람* : 미이탈율 10711 / 13657, 78%, 미이탈 유저중 54.6%, 이탈 유저들의 수의 총량도 매우
작다.
