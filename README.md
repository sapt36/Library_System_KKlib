# KKday 實習 – kklib SDK 練習專案
![image](https://github.com/user-attachments/assets/a5c38240-8dd8-4b74-8328-d70f4bc30710)
---

## 專案簡介

本專案為我在 KKday 擔任後端實習生的 Spring Boot 練習專案，從中學習如何整合 **KKday SDK**、PostgreSQL、RabbitMQ、Redis 及多環境部署腳本，並透過 Woodpecker CI/CodeDeploy 完成自動化建置、測試與佈署。

主要涵蓋以下主題：

| 分類     | 內容                                               |
| ------ | ------------------------------------------------ |
| Domain | `Book`、`User` CRUD REST API                      |
| 外部介接   | Fake API (書籍 / 郵件)                               |
| 非同步    | MQ 事件 (`BookMQ*`)、背景執行緒                          |
| 基礎建設   | Flyway 資料庫版本控管、Ehcache、Logback                   |
| DevOps | Woodpecker Pipeline、AWS CodeDeploy `appspec.yml` |

---

## 專案結構

```
.
├── src
│   ├── main
│   │   ├── java/com/kkday/svc/kklib        ← 業務 & SDK 整合程式碼
│   │   └── resources                       ← 設定檔／模板／SQL
│   └── test                                ← 測試與 Mock 資料
├── scripts                                 ← 部署啟停腳本（CodeDeploy）
├── config                                  ← Static analysis / License 檔
├── gradle* / gradlew*                      ← Build Tool (Gradle 8.5 Wrapper)
├── .woodpecker.yml                         ← CI Pipeline 定義
└── appspec.yml                             ← CodeDeploy 佈署描述
```

---

## 技術與相依

| 類別        | 技術 / 版本                      | 備註                             |
| --------- | ---------------------------- | ------------------------------ |
| JVM       | **Java 17+**                 | `toolchain` 可自訂                |
| Framework | Spring Boot 3.x<br>KKday SDK | REST / MQ / Redis / Tx / Lock  |
| Build     | Gradle 8.5                   | Wrapper 已內建                    |
| DB        | PostgreSQL 15                | `Flyway` 管理版本                  |
| Cache     | Redis 7                      | 可關閉 via `application.yml`      |
| MQ        | RabbitMQ 3                   | Prefix 與佇列於 `application*.yml` |
| Lint      | SpotBugs、PMD                 | CI 自動執行                        |
| Test      | JUnit 5、Mockito              | Profile `test` 使用 H2           |

---

## 開發環境快速啟動

1. **安裝相依**

   ```bash
   # macOS / Linux 建議
   brew install openjdk@17 docker docker-compose
   ```
2. **啟動服務 (Docker)**

   ```bash
   docker compose -f docker/local-stack.yml up -d   # PostgreSQL / Redis / RabbitMQ
   ```
3. **執行應用 (Local Profile)**

   ```bash
   ./gradlew bootRun --args='--spring.profiles.active=local'
   ```
4. **確認 Swagger**  ➜  [http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)

---

## 主要指令

| 指令                                   | 說明                              |
| ------------------------------------ | ------------------------------- |
| `./gradlew clean build`              | 編譯並執行單元測試                       |
| `./gradlew flywayMigrate`            | 依 `flyway.conf` 執行 DB Migration |
| `./gradlew spotbugsMain` / `pmdMain` | 靜態程式碼掃描                         |
| `./gradlew bootJar`                  | 產出可執行 Jar                       |

---

## CI / CD 流程

* **Woodpecker CI**（`.woodpecker.yml`）在 Pull Request 觸發：

  1. 編譯 & 單元測試
  2. SpotBugs / PMD / License Check
* **GitHub Release Helper**：自動產生 `CHANGELOG`（`kklib_v*` 標籤）
* **AWS CodeDeploy**：使用 `appspec.yml` 與 `scripts/` 完成藍綠佈署

---

## 配置檔說明

| 檔案                               | 用途                          |
| -------------------------------- | --------------------------- |
| `application.yml`                | 共用設定（預設 dev）                |
| `application-local.yml`          | 本地開發 Profile                |
| `application-cloud.yml.template` | 雲端環境範本，參數由 CI 注入            |
| `service-*.yml`                  | SDK 延伸設定 (JPA / Redis / MQ) |

> **TIP**：可藉由 `--spring.profiles.active=<profile>` 切換環境。

---

## 資料庫版本控管 (Flyway)

* SQL 檔位於 `sql/`，檔名格式 `V<版本號>__<描述>.sql`
* 執行

  ```bash
  ./gradlew flywayMigrate -Dflyway.configFiles=flyway.conf
  ```

---

## 測試

```bash
./gradlew test                      # 單元 + 整合測試
./gradlew jacocoTestReport          # 產生覆蓋率報告
```

測試設定檔位於 `src/test/resources/config/`，H2 記憶體資料庫自動啟動。

---

## 常見問題

| 問題                     | 解法                                                                 |
| ---------------------- | ------------------------------------------------------------------ |
| `JAVA_HOME is not set` | `export JAVA_HOME=$(dirname $(dirname $(readlink $(which java))))` |
| 無法連線 DB                | 確認 `application-local.yml` 與 Docker PG 容器連線資訊                      |
| MQ 佇列不存在               | 啟動 RabbitMQ 或修改 `mq.prefix` 設定                                     |

---

## 授權

本專案採用 **Apache License 2.0**。
