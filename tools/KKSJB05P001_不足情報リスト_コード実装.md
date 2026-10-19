# KKSJB05P001_不足情報リスト_コード実装

対象：KKSJB05P001_不足情報リスト（2026/10/19 作成）の各項目について、実際に納品された Java コードではどう実装されているかを対応付けたもの
作成日：2026/10/20

## 0. 目的と読み方

- 本書は、不足情報リストに挙げた「仕様書に名前はあるが定義がない」項目について、実コード（NRI 作成の実装）で確認できた実装内容を、項目ごとに対応付けたものである。不足情報リスト本体は変更していない。本書はその対になる資料である。
- 実装の根拠は、写真から転記した次の6ファイルだけである。6ファイルに写っていないものは、本書でも「未判明」と明記する。推測で補った項目はない。
- KKSJB05P001JobConfig.java（193行、パッケージ jp.or.ja_kyosai.wmp.apl.job.config）
- KKSJB05P001Reader.java（313行、パッケージ jp.or.ja_kyosai.wmp.apl.job.reader）
- KKSJB05P001Processor.java（265行、パッケージ jp.or.ja_kyosai.wmp.apl.job.processor）
- KKSJB05P001Writer.java（189行、パッケージ jp.or.ja_kyosai.wmp.apl.job.writer）
- KKSJB05P001InDto.java（19行、パッケージ jp.or.ja_kyosai.wmp.apl.job.dto）
- KKSJB05P001ProOutDto.java（21行、パッケージ jp.or.ja_kyosai.wmp.apl.job.dto）
- コード抜粋はすべて転記ファイルからの原文のままで、行番号は各ファイルの行番号である。
- 判明状況は次の3区分で示す。
- 「定義本体まで判明」：そのクラス自体のソースが写真で確認できた。
- 「呼出側の実装で判明」：定義本体は写真にないが、6ファイル内の呼出コードから、パッケージ・メソッド名・引数・戻り値・定数名などが確定できた。
- 「依然として未判明」：6ファイルからは分からない。追加で必要な資料を明記する。

### 対応表（節ごとの判明状況）

| 節 | 項目 | 判明状況 |
|---|---|---|
| 1 | DTO／Entity のクラス定義（9クラス） | InDto／ProOutDto は定義本体まで判明。他の7クラスはパッケージと、呼出側で使われる getter／setter まで判明。定義本体は未判明 |
| 2 | DAO のクラス定義と SQL（4クラス、7メソッド） | 全7メソッドの取得方法・引数・戻り値が呼出側の実装で判明。SQL 本体と Mapper 定義は依然として未判明 |
| 3 | 定数（29件） | 全29件の実装上の定数名が判明。値まで判明したのは JobInfConstants の分（#1 と、29件外の StepID）で、CommonConstants の値は未判明（定数クラスの定義本体が未送付のため） |
| 4 | Enum の定義（4種） | 4種ともクラス名・パッケージ・メンバー名が判明。getValue() が返すコード値は未判明 |
| 5 | メッセージ定義（8件） | メッセージID に対応する Enum メンバー名、呼出箇所、ログパラメータが判明。メッセージ文言そのものは未判明 |
| 6 | 共通部品・フレームワーク側の定義（11項目） | 11項目すべて、提供クラスと呼出方法が判明。部品内部の実装仕様（チェック内容そのものなど）は未判明 |
| 7 | テーブル定義とシーケンス（5件） | アクセス経路（どの DAO メソッドがどのテーブル・シーケンスに対応するか）のみ判明。列定義・シーケンス定義は未判明 |
| 8 | プロジェクト規約（5項目） | パッケージ、アノテーション、DTO の getter／setter 規約、ジョブ定義方式（Java Config）が判明。バージョンは未判明 |

## 1. DTO／Entity のクラス定義（9クラス）

### 1-1 KKSJB05P001InDto

ファイル全体が写真で確認できた。実装は次のとおり（KKSJB05P001InDto.java 全19行）。

```java
package jp.or.ja_kyosai.wmp.apl.job.dto;

import java.io.Serializable;

import jp.or.ja_kyosai.wmp.apl.job.dto.common.CustomercopyinfoDto;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class KKSJB05P001InDto implements Serializable {

private static final long serialVersionUID = 1L;

private CustomercopyinfoDto customercopyinfoDto;  // 利用者控情報管理DTO
private String mypageId;  // マイページID
private String holderMailAdress;  // 契約者メールアドレス

}
```

- パッケージは jp.or.ja_kyosai.wmp.apl.job.dto。
- getter／setter は手書きではなく **Lombok の @Getter／@Setter**（クラスレベル）で生成する。
- 項目の英名は、customercopyinfoDto（利用者控情報管理DTO）、mypageId（マイページID）、holderMailAdress（契約者メールアドレス）。holderMailAdress は "Adress" と d が1文字欠けた綴りがフィールド名そのもので、Processor／Reader の getHolderMailAdress() はこのフィールド名から Lombok が生成したものである。
- implements Serializable で、serialVersionUID = 1L。

### 1-2 KKSJB05P001ProOutDto

ファイル全体が写真で確認できた。実装は次のとおり（KKSJB05P001ProOutDto.java 全21行）。

```java
package jp.or.ja_kyosai.wmp.apl.job.dto;

import java.io.Serializable;

import jp.or.ja_kyosai.wmp.apl.job.dto.common.CustomercopyinfoDto;
import jp.or.ja_kyosai.wmp.apl.job.dto.common.MailsendDto;
import jp.or.ja_kyosai.wmp.apl.job.dto.common.SmssendDto;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class KKSJB05P001ProOutDto implements Serializable {

private static final long serialVersionUID = 1L;

private MailsendDto mailsendDto;  // メール送付情報DTO
private SmssendDto smssendDto;  // SMS送付情報DTO
private CustomercopyinfoDto customercopyinfoDto;  // 利用者控情報管理DTO

}
```

- パッケージは jp.or.ja_kyosai.wmp.apl.job.dto。Lombok、Serializable、serialVersionUID の扱いは InDto と同じ。
- 保持する3つの DTO はいずれも jp.or.ja_kyosai.wmp.apl.job.dto.common 配下である。

### 1-3 CustomercopyinfoDto

- パッケージ: jp.or.ja_kyosai.wmp.apl.job.dto.common（Reader／Processor の import で確定）。
- クラス定義本体は写真にない。ただし呼出側のコードで使われる getter／setter から、仕様書の15項目すべての英名が確定した。

| 仕様書の項目（和名） | 英名（getter／setter から確定） | 主な使用箇所 |
|---|---|---|
| 県番号 | prefectureNo | Reader convertToDto、Processor updateCustomercopyinfo |
| レコード保有情報区分 | recordHoldInformationCls | Reader convertToDto、Processor updateCustomercopyinfo |
| 控え情報管理枝番 | copyInformationMngBranchNo | Reader convertToDto、Processor updateCustomercopyinfo |
| ジャーナル種類名 | journalTypeName | Reader convertToDto、Processor getMailsend／getSmssend |
| ジャーナル用利用日時 | journalUseDatetime | Reader convertToDto、Processor process／getMailsend／getSmssend |
| 収納委託番号 | rcpRequestNo | Reader convertToDto、Processor updateCustomercopyinfo |
| 権利譲渡日時 | authorityDatetime | Reader convertToDto、Processor process |
| 申込管理番号 | applicationMngNo | Reader convertToDto（002側のみ） |
| 申込管理番号枝番 | applicationMngBranchNo | Reader convertToDto（002側のみ） |
| 不変番号 | invariantNo | Reader convertToDto（001側のみ）、Writer の更新エラーログ |
| メール作成ステータス | mailMakeSts | Processor updateCustomercopyinfo |
| 表示フラグ | displayFlg | Processor updateCustomercopyinfo |
| バッチ更新日 | batchUpdateDay | Processor updateCustomercopyinfo |
| バッチ更新時刻 | batchUpdateTime | Processor updateCustomercopyinfo |
| バッチ更新プログラムID | batchUpdateProgramId | Processor updateCustomercopyinfo |

- 設定側の実装（KKSJB05P001Reader.java 253～263行目、convertToDto の001側）:

```java
dto.setMypageId(entity001.getMypageId());
dto.setHolderMailAdress(entity001.getHolderMailAdress());
copyDto.setPrefectureNo(entity001.getPrefectureNo());
copyDto.setRecordHoldInformationCls(
entity001.getRecordHoldInformationCls());
copyDto.setCopyInformationMngBranchNo(
entity001.getCopyInformationMngBranchNo());
copyDto.setJournalTypeName(entity001.getJournalTypeName());
copyDto.setJournalUseDatetime(entity001.getJournalUseDatetime());
copyDto.setRcpRequestNo(entity001.getRcpRequestNo());
copyDto.setAuthorityDatetime(entity001.getAuthorityDatetime());
copyDto.setInvariantNo(entity001.getInvariantNo());
```

- 002側では setInvariantNo の代わりに setApplicationMngNo／setApplicationMngBranchNo が使われる（同ファイル 281～282行目）。
- 呼出コードから読み取れる型の手掛かり: getJournalUseDatetime() と getAuthorityDatetime() の戻り値は、Processor の process で SimpleDateFormat.parse の引数になっているため String である（KKSJB05P001Processor.java 51～58行目）。他の項目の型は定義本体がないため確定できない。

### 1-4 MailsendDto

- パッケージ: jp.or.ja_kyosai.wmp.apl.job.dto.common。
- Processor の getMailsend で全項目が設定される。設定の実装（KKSJB05P001Processor.java 130～148行目）:

```java
mailsendDto.setOnlineUpdateDay(null);
mailsendDto.setOnlineUpdateTime(null);
mailsendDto.setOnlineUpdateUserId(null);
mailsendDto.setOnlineUpdateProgramId(null);
mailsendDto.setBatchUpdateDay(null);
mailsendDto.setBatchUpdateTime(null);
mailsendDto.setBatchUpdateProgramId(JobInfConstants.JobID.KKSJB05P001);
mailsendDto.setCreateDay(null);
mailsendDto.setCreateTime(null);
mailsendDto.setCreateUserId(CommonConstants.SYS_ID);
mailsendDto.setCreateProgramId(JobInfConstants.JobID.KKSJB05P001);
mailsendDto.setMypageId(inEntity.getMypageId());
mailsendDto.setMailTransmitSequence(null);
mailsendDto.setMailCls(CommonConstants.MAIL_CLS_E2);
mailsendDto.setMailAddMaintext(mailAddMaintext);
mailsendDto.setMailTransmitStatus(CommonConstants.MAIL_TRANSMIT_STATUS_0);
```

- 仕様書の16項目との対応: マイページID＝mypageId、メール送付シーケンス＝mailTransmitSequence、メール区分＝mailCls、メール追加本文＝mailAddMaintext、メール送付ステータス＝mailTransmitStatus、生成日＝createDay、生成時刻＝createTime、生成ユーザID＝createUserId、生成プログラムID＝createProgramId、オンライン更新日＝onlineUpdateDay、オンライン更新時刻＝onlineUpdateTime、オンライン更新ユーザID＝onlineUpdateUserId、オンライン更新プログラムID＝onlineUpdateProgramId、バッチ更新日＝batchUpdateDay、バッチ更新時刻＝batchUpdateTime、バッチ更新プログラムID＝batchUpdateProgramId。16項目すべて英名が確定した。
- メール送付シーケンスは Processor では null が設定され、採番後の値は Writer で setMailTransmitSequence(mailsendSeq) により設定される（KKSJB05P001Writer.java 79行目）。
- メール追加本文の組み立ては StringBuilder で、定数 SINGLEQUOTE と COMMA を使う（KKSJB05P001Processor.java 121～126行目）。3の定数も参照。

### 1-5 SmssendDto

- パッケージ: jp.or.ja_kyosai.wmp.apl.job.dto.common。
- Processor の getSmssend で設定される。設定の実装（KKSJB05P001Processor.java 178～200行目）:

```java
smssendDto.setOnlineUpdateDay(null);
smssendDto.setOnlineUpdateTime(null);
smssendDto.setOnlineUpdateUserId(null);
smssendDto.setOnlineUpdateProgramId(null);
smssendDto.setBatchUpdateDay(null);
smssendDto.setBatchUpdateTime(null);
smssendDto.setBatchUpdateProgramId(JobInfConstants.JobID.KKSJB05P001);
smssendDto.setCreateDay(null);
smssendDto.setCreateTime(null);
smssendDto.setCreateUserId(CommonConstants.SYS_ID);
smssendDto.setCreateProgramId(JobInfConstants.JobID.KKSJB05P001);
smssendDto.setMypageId(inEntity.getMypageId());
smssendDto.setMailCls(CommonConstants.MAIL_CLS_E3);
smssendDto.setSmsAddMaintext(smsAddMaintext);
smssendDto.setSmsTransmitStatus(CommonConstants.SMS_SEND_STATUS_0);
smssendDto.setSmsSid(null);
smssendDto.setSmsTransmitNumber(
new java.math.BigDecimal(CommonConstants.INT_ZERO));
smssendDto.setLastSendTime(null);
smssendDto.setResultStatus(CommonConstants.RESULT_STATUS_0);
smssendDto.setLastResultTime(null);
smssendDto.setErrorCode(null);
```

- 仕様書の21項目との対応: マイページID＝mypageId、SMS_SID＝smsSid、メール区分＝mailCls、SMS追加本文＝smsAddMaintext、前回送信日時＝lastSendTime、SMS送信回数＝smsTransmitNumber、前回結果取得日時＝lastResultTime、結果取得ステータス＝resultStatus、SMS送付ステータス＝smsTransmitStatus、エラーコード＝errorCode、生成日＝createDay、生成時刻＝createTime、生成ユーザID＝createUserId、生成プログラムID＝createProgramId、オンライン更新日＝onlineUpdateDay、オンライン更新時刻＝onlineUpdateTime、オンライン更新ユーザID＝onlineUpdateUserId、オンライン更新プログラムID＝onlineUpdateProgramId、バッチ更新日＝batchUpdateDay、バッチ更新時刻＝batchUpdateTime、バッチ更新プログラムID＝batchUpdateProgramId。21項目すべて英名が確定した。
- SMS送信回数（smsTransmitNumber）の型は java.math.BigDecimal で、import せず完全修飾名で new している。
- 仕様書の21項目にはない SMS送付シーケンス（smsTransmitSequence）も、Writer で setSmsTransmitSequence(smssendSeq) により設定される（KKSJB05P001Writer.java 115行目）。

### 1-6 KIYKB477001OutEntity

- パッケージ: jp.or.ja_kyosai.wmp.apl.mybatis.entity（Reader の import で確定）。
- クラス定義本体は写真にない。Reader の convertToDto で使われる getter から、仕様書の10項目すべての英名が確定した: マイページID＝mypageId、契約者メールアドレス＝holderMailAdress、県番号＝prefectureNo、レコード保有情報区分＝recordHoldInformationCls、控え情報管理枝番＝copyInformationMngBranchNo、ジャーナル種類名＝journalTypeName、ジャーナル用利用日時＝journalUseDatetime、収納委託番号＝rcpRequestNo、権利譲渡日時＝authorityDatetime、不変番号＝invariantNo（設定側の抜粋は 1-3 のコードのとおり）。

### 1-7 KIYKB477002OutEntity

- パッケージ: jp.or.ja_kyosai.wmp.apl.mybatis.entity。
- 001との差分はコードでもそのまま現れている: 不変番号（getInvariantNo）はなく、申込管理番号＝applicationMngNo、申込管理番号枝番＝applicationMngBranchNo がある。共通の9項目（マイページID、契約者メールアドレス、県番号、レコード保有情報区分、控え情報管理枝番、ジャーナル種類名、ジャーナル用利用日時、収納委託番号、権利譲渡日時）の英名は001と同じ。仕様書の11項目すべて英名が確定した。

### 1-8 KIYKB477003InEntity

- パッケージ: jp.or.ja_kyosai.wmp.apl.mybatis.entity。
- 使われ方は Writer の更新処理のみ（KKSJB05P001Writer.java 150～159行目）: new で生成し、利用者控情報管理DTO から CommonUtils.copyBeanToBean でコピーし、KIYKB477DbDao.update001 に渡す。

```java
updateEntity = new KIYKB477003InEntity();
// 2-3-1-2. 変数. 利用者控情報管理更新エンティティ作成
CommonUtils.copyBeanToBean(outDto.getCustomercopyinfoDto(), updateEntity);
```

- 項目定義はコピー元の CustomercopyinfoDto と同名項目でコピーされる前提だが、このクラス自体の項目一覧は6ファイルからは確定できない。

### 1-9 MailsendEntity／SmssendEntity

- パッケージ: いずれも jp.or.ja_kyosai.wmp.apl.mybatis.entity。
- 使われ方は Writer の登録処理（KKSJB05P001Writer.java 81～90行目、117～126行目）: new で生成し、DTO から CommonUtils.copyBeanToBean でコピーし、各 DAO の insert に渡す。

```java
mailsendEntity = new MailsendEntity();
// メール送付情報エンティティデータ作成
CommonUtils.copyBeanToBean(outDto.getMailsendDto(), mailsendEntity);
```

- MailsendEntity で直接参照される getter は、登録エラーログのパラメータとして使われる getMypageId() と getMailTransmitSequence() の2件（同ファイル 97～101行目）。
- SmssendEntity の getter は6ファイル内では直接呼ばれていない（SMS の登録エラーログは DTO 側の getMypageId()／getSmsTransmitSequence() を使っている。同ファイル 133～139行目）。両 Entity の項目一覧は定義本体がないため確定できない。

## 2. DAO のクラス定義と SQL（4クラス、7メソッド）

### 共通する実装方式

- DAO のパッケージは jp.or.ja_kyosai.wmp.apl.mybatis.dao（Reader／Writer の import で確定）。
- 仕様書では生成方法が確定できなかったが、実コードでは全箇所とも **getMapper(DAOクラス.class)** で Mapper インスタンスを取得する方式に統一されている。getMapper は Reader／Writer のベースクラス（CustomDbItemReader／CustomDbItemWriter）が提供するメソッドである。仕様書の入力処理02 にあった「new KIYKB477DbDao()」という記述は、実コードでは使われていない。
- 呼出側から確定した7メソッドのシグネチャは次のとおり。

| # | クラス | メソッド | 引数 | 戻り値 | 呼出箇所 |
|---|---|---|---|---|---|
| 1 | KIYKB477DbDao | select001 | なし | List\<KIYKB477001OutEntity\> | Reader 95行目 |
| 2 | KIYKB477DbDao | select002 | なし | List\<KIYKB477002OutEntity\> | Reader 123行目 |
| 3 | KIYKB477DbDao | update001 | KIYKB477003InEntity | 呼出側では受けていない（文として呼出） | Writer 159行目 |
| 4 | MailsendBtTblDbDao | insert | MailsendEntity | 呼出側では受けていない（文として呼出） | Writer 90行目 |
| 5 | SmssendBtTblDbDao | insert | SmssendEntity | 呼出側では受けていない（文として呼出） | Writer 126行目 |
| 6 | MbComnSeqDbDao | selectSeq003 | なし | String（メール送付シーケンス） | Writer 77行目 |
| 7 | MbComnSeqDbDao | selectSeq006 | なし | String（SMS送付シーケンス） | Writer 113行目 |

### 2-1 select001／select002（KIYKB477DbDao）

KKSJB05P001Reader.java 92～95行目:

```java
// 2-1-1. 変数. 利用者控連携DAOのMapperインスタンスを取得する。
KIYKB477DbDao kiykb477DbDao = getMapper(KIYKB477DbDao.class);
// 2-1-2. 不変番号に紐づく利用者控およびマイページIDの取得の実行
kIYKB477001OutEntityList = kiykb477DbDao.select001();
```

select002 も同じ形で、戻り値はフィールド kIYKB477002OutEntityList に受ける（同ファイル 120～123行目）。引数はどちらもなく、抽出条件は SQL 側に持つ形である。

### 2-2 update001（KIYKB477DbDao）

KKSJB05P001Writer.java 156～159行目:

```java
// 変数. 利用者控連携DAOのMapperインスタンスを取得する。
KIYKB477DbDao kiykb477DbDao = getMapper(KIYKB477DbDao.class);
// 利用者控情報管理更新の実行
kiykb477DbDao.update001(updateEntity);
```

### 2-3 insert（MailsendBtTblDbDao／SmssendBtTblDbDao）

KKSJB05P001Writer.java 86～90行目（メール側）:

```java
// 変数. メール送付情報DAOのMapperインスタンスを取得する。
MailsendBtTblDbDao mailsendBtTblDbDao = getMapper(
MailsendBtTblDbDao.class);
// メール送付情報登録の実行
mailsendBtTblDbDao.insert(mailsendEntity);
```

SMS側も同じ形（同ファイル 122～126行目、SmssendBtTblDbDao／smssendEntity）。

### 2-4 selectSeq003／selectSeq006（MbComnSeqDbDao）

KKSJB05P001Writer.java 75～77行目（メール側）:

```java
MbComnSeqDbDao mbComnSeqDbDao = getMapper(MbComnSeqDbDao.class);
// メール送付シーケンスを取得する。
mailsendSeq = mbComnSeqDbDao.selectSeq003();
```

SMS側は selectSeq006（同ファイル 111～113行目）。取得した String は、そのまま DTO の setMailTransmitSequence／setSmsTransmitSequence に設定される。

### 2-5 SQL 本体と Mapper 定義

- 7メソッドすべての **SQL 本体（抽出条件、更新項目、登録項目、採番の SQL）は6ファイルに含まれていない**。DAO はインターフェースとしてのみ現れ、SQL は MyBatis の Mapper 定義（XML またはアノテーション）側にあるため、呼出側のコードからは読み取れない。
- DAO インターフェースの定義本体（メソッド宣言の正確な形、戻り値の型宣言）も未送付である。本節のシグネチャは呼出側の使い方から確定した範囲である。
- 必要な資料: DAO の Mapper 定義一式（KIYKB477DbDao、MailsendBtTblDbDao、SmssendBtTblDbDao、MbComnSeqDbDao に対応するもの）。

## 3. 定数（29件）

### 判明の範囲

- 定数は2クラスに分かれている: **CommonConstants**（jp.or.ja_kyosai.wmp.apl.common.constants）と **JobInfConstants**（同パッケージ。ジョブID・ステップID は内部クラス JobID／StepID の定数）。
- 不足情報リストの29件すべてについて、実装上の定数名が確定した。
- 一方、**定数の値そのものは6ファイルからは分からない**（定数クラスの定義本体が未送付）。例外として、ログの固定文字（#20～#28）は値である文字列そのものが仕様書で判明済みで、本書で定数名が対応付いた。

| # | 定数（仕様書の表記） | 実装（定数名） | 使われ方・出現箇所 |
|---|---|---|---|
| 1 | ジョブID_KKSJB05P001 | JobInfConstants.JobID.KKSJB05P001 | Config の @Bean 名と jobBuilderFactory.get の引数（137、142行目）。Processor では生成プログラムID／バッチ更新プログラムID の設定値としても使われる。値は **"KKSJB05P001"**（String、JobInfConstants.java 228行目。2026/10/20 の写真で確定） |
| 2 | 検索件数 | CommonConstants.COM_STR_SEARCH_CNT | ジョブコンテクストのキー。Reader の setJobContext（148行目）、Config の getJobContextForInt（86行目） |
| 3 | 空白 | CommonConstants.BLANK | Processor の権利譲渡判定フラグの初期値（46行目）、Writer のエラーコードの初期値（68行目） |
| 4 | コンマ | CommonConstants.COMMA | Processor のメール／SMS追加本文の組み立て（123、171行目） |
| 5 | シングルクオート | CommonConstants.SINGLEQUOTE | Processor のメール／SMS追加本文の組み立て（121～126、169～174行目） |
| 6 | パイプ | CommonConstants.PIPE | Reader の merge で重複チェックキーの連結（179～185、211～215行目） |
| 7 | 整数0 | CommonConstants.INT_ZERO | Reader の merge の件数判定（164、199行目）、Processor の SMS送信回数の初期値 new BigDecimal(CommonConstants.INT_ZERO)（194～195行目） |
| 8 | SYS_ID | CommonConstants.SYS_ID | Processor の生成ユーザID の設定値（getMailsend 138行目、getSmssend 186行目） |
| 9 | メール作成ステータス「未作成」（値 "0"） | コード内では参照されない | 抽出条件として select001／select002 の SQL 側にある形で、6ファイルには現れない |
| 10 | メール作成ステータス「作成済み」 | CommonConstants.MAIL_MAKE_STS_1 | Processor updateCustomercopyinfo で、設定条件（マイページIDが非空 かつ 権利譲渡ありではない）が仕様書の「作成済み」の設定条件に対応する側（244行目付近）。定数値そのものは未判明 |
| 11 | メール作成ステータス「処理対象外」 | CommonConstants.MAIL_MAKE_STS_2 | 同メソッドで上記以外の場合に設定される側（248行目付近）。定数値そのものは未判明 |
| 12 | メール送付ステータス「未送信」 | CommonConstants.MAIL_TRANSMIT_STATUS_0 | Processor getMailsend の setMailTransmitStatus（148行目） |
| 13 | SMS送付ステータス「未送信」 | CommonConstants.SMS_SEND_STATUS_0 | Processor getSmssend の setSmsTransmitStatus（191行目） |
| 14 | メール区分「利用者控連携完了通知メール」 | CommonConstants.MAIL_CLS_E2 | Processor getMailsend の setMailCls（145行目） |
| 15 | メール区分「利用者控連携完了通知SMS」 | CommonConstants.MAIL_CLS_E3 | Processor getSmssend の setMailCls（189行目） |
| 16 | 結果取得ステータス「未取得」 | CommonConstants.RESULT_STATUS_0 | Processor getSmssend の setResultStatus（198行目） |
| 17 | 権利譲渡あり | CommonConstants.AUTHORITY_ASSIGNMENT_CLS_1 | Processor process で権利譲渡判定フラグへの設定（62行目）と、判定条件での比較（78、245、254行目） |
| 18 | 表示フラグ「表示」 | CommonConstants.DISPLAY_FLG_0／DISPLAY_FLG_1 のいずれか | Processor updateCustomercopyinfo で、権利譲渡ありの場合は DISPLAY_FLG_0、それ以外は DISPLAY_FLG_1 が設定される（253～259行目）。「表示」「非表示」のどちらが 0 かは、定数値が未判明のため確定できない |
| 19 | 表示フラグ「非表示」 | 同上（#18 と対） | 同上 |
| 20 | ログの固定文字「利用者控情報管理」 | CommonConstants.COM_LOG_CONTENT_305 | 異常系ログ（MBKKSJB0020E／MBKKSJB0060E）の第1パラメータ、および件数ログ（MBKKSJB0027I）の第1パラメータに広く使われる |
| 21 | ログの固定文字「データ読込件数」 | CommonConstants.COM_LOG_CONTENT_33 | Config afterJob の読込件数ログの第2パラメータ（92行目） |
| 22 | ログの固定文字「メール送付情報」 | CommonConstants.MAILSEND_INFO | Config afterJob の登録件数ログの第1パラメータ（102行目）、Writer のメール登録エラーログの第1パラメータ（99行目） |
| 23 | ログの固定文字「メール送付情報登録件数」 | CommonConstants.COM_LOG_CONTENT_106 | Config afterJob の登録件数ログの第2パラメータ（103行目） |
| 24 | ログの固定文字「SMS送付情報」 | CommonConstants.COM_LOG_CONTENT_3_45 | Config afterJob の登録件数ログの第1パラメータ（113行目）、Writer の SMS登録エラーログの第1パラメータ（135行目） |
| 25 | ログの固定文字「SMS送付情報登録件数」 | CommonConstants.COM_LOG_CONTENT_3_60 | Config afterJob の登録件数ログの第2パラメータ（114行目） |
| 26 | ログの固定文字「利用者控情報管理更新件数」 | CommonConstants.CUSTOMERCOPYINFO_UPDATE_CNT | Config afterJob の更新件数ログの第2パラメータ（125行目）。4の CountEnum.COUNT_CUSTOMERCOPYINFOUPDCNT とは別の定数で、こちらはログパラメータ用である |
| 27 | ログの固定文字「マイページ保有契約」 | CommonConstants.MYPAG_HOYU_KEIYAKU | Reader の select001 側の検索ログの第1パラメータ（103行目） |
| 28 | ログの固定文字「マイページ仮登録保有契約」 | CommonConstants.MYPAG_TMP_HOYU_KEIYAKU | Reader の select002 側の検索ログの第1パラメータ（131行目）。#27 と異なり TMP が入る |
| 29 | 固定文言「PHMFダミーアドレス(フル)」 | CommonConstants.PHMF_DUMMY_ADDRESS_FULL | Processor process の SMS作成対象の判定条件（89～91行目） |

### 29件以外にコードで判明した定数

- JobInfConstants.StepID.KKSJB05P001_S01: ステップ名。Config の stepBuilderFactory.get の引数（149行目）。値は **"KKSJB05P001S01"**（String。定数名と異なり、値には P001 と S01 の間のアンダースコアが入らない。2026/10/20 の写真で確定）。
- CommonConstants.ENCODING_UTF8: Config の @PropertySource の encoding 指定（39～40行目）。
- ALLOW_STARTIF_COMPLETE と CHUNK_COUNT_KEY: Config 内で修飾なしに使われており、ベースクラス AbstractJobBase から継承した定数である（155、158行目）。それぞれ仕様書のプロパティ job.allowStartIfComplete、job.commitInterval に対応するキーとして使われる（8-5 参照）。

### 依然として未判明

- CommonConstants の各定数の値と、両定数クラスの定義本体。JobInfConstants については、本ジョブで使う2件（JobID.KKSJB05P001、StepID.KKSJB05P001_S01）の値が写真で確定した。必要な資料: 定数一覧、または両クラスのソース。
- なお、2026/10/20 の検索結果の写真では、JobInfConstants.java 内に KKSJB05P001 の定義行らしきものが2件、KKSJB05P001_S01 の定義行らしきものが2件見えた。別の内部クラスにも同名の定数がある可能性があるが、写真の範囲では確定できない。

## 4. Enum の定義（4種）

4種とも実クラス名とパッケージが確定した。いずれもメンバーを参照する際は Enum 定数そのものか、getValue() の戻り値（キー文字列）を使う形である。getValue() が返す値そのものは、Enum の定義本体がないため未判明である。

### 4-1 ログ種別Enum → LogKindEnum

- 実クラス: jp.or.ja_kyosai.wmp.apl.common.log.LogKindEnum（他の Enum と異なり、constants.enums ではなく common.log 配下）。
- 仕様書の5メンバーすべてに対応する実装名が確定した。

| 仕様書のメンバー | 実装（LogKindEnum） | 出現箇所 |
|---|---|---|
| プログレス | PROGRESS | Config afterJob の件数ログ4件 |
| キャッチ | CATCH_ON_EXHANDLER | Config afterJob の DBアクセスエラーログ |
| テーブル検索 | TABLE_SELECT | Reader init の検索ログと、その例外時ログ |
| テーブル登録 | TABLE_INSERT | Writer write の登録エラーログ2件 |
| テーブル更新 | TABLE_UPDATE | Writer write の更新エラーログ |

### 4-2 メッセージＩＤEnum → MsgIdEnum

- 実クラス: jp.or.ja_kyosai.wmp.apl.common.constants.enums.MsgIdEnum。
- 参照形式は MsgIdEnum.MSGID_＜メッセージID＞.getValue() で、logWriter の第2引数に渡す。メンバー名とメッセージIDの対応、呼出箇所は5にまとめた。

### 4-3 テーブルＩＤEnum → TableIdEnum

- 実クラス: jp.or.ja_kyosai.wmp.apl.common.constants.enums.TableIdEnum。
- 確定したメンバーは2件: メール送付情報TBL＝TBLID_MAILSEND、SMS送付情報TBL＝TBLID_SMSSEND。ジョブコンテクストのキーとして、TableIdEnum.TBLID_MAILSEND.getValue() の形で使う（Writer 179、182行目／Config 98、109行目）。
- 仕様書の3件目「利用者控情報管理TBL」に対応するメンバーは、6ファイルには現れない。Writer の該当箇所は、コメントは「テーブルＩＤEnum. 利用者控情報管理TBL」と書かれているが、コードは次のとおり処理件数Enum（CountEnum）を使っている（KKSJB05P001Writer.java 184～186行目）。

```java
// 3-3. ジョブコンテクスト{テーブルＩＤEnum. 利用者控情報管理TBL}に、クラス変数. 利用者控情報管理更新件数を
setJobContext(CountEnum.COUNT_CUSTOMERCOPYINFOUPDCNT.getValue(),
customercopyinfoUpdateCnt);
```

### 4-4 処理件数Enum → CountEnum

- 実クラス: jp.or.ja_kyosai.wmp.apl.common.constants.enums.CountEnum。
- 確定したメンバーは1件: 利用者控情報管理の更新件数＝COUNT_CUSTOMERCOPYINFOUPDCNT。Writer が setJobContext で設定し（185行目）、Config が getJobContextForInt で取得する（119～120行目）。他のメンバーの有無は定義本体がないため不明である。

## 5. メッセージ定義（8件）

メッセージ文言（本文）とプレースホルダの仕様は、6ファイルのどこにも現れないため **依然として未判明** である（必要な資料: メッセージ定義書）。コードで確定したのは、各ID の MsgIdEnum メンバー名、ログ出力の呼出箇所、ログ種別、パラメータ配列の中身である。

| # | メッセージID | MsgIdEnum メンバー | 呼出箇所とパラメータ |
|---|---|---|---|
| 1 | MBBASEB0001I | 6ファイル内では未使用 | 開始ログ。仕様書で「ベースクラスで実装する」とされた範囲で、Config の beforeJob にも出力コードはない |
| 2 | MBBASEB0002I | 6ファイル内では未使用 | 終了ログ。同上（Config の afterJob に出力コードはない） |
| 3 | MBBASEB0004E | 6ファイル内では未使用 | 異常終了ログ。同上 |
| 4 | MBKKSJB0020E | MSGID_MMBKKSJB0020E（MB の前に M が1つ多く、他の4件と表記が非対称。原文のまま） | Config afterJob（ログ種別 CATCH_ON_EXHANDLER）と Reader init の例外時（ログ種別 TABLE_SELECT）の2系統。パラメータはどちらも { CommonConstants.COM_LOG_CONTENT_305} |
| 5 | MBKKSJB0027I | MSGID_MBKKSJB0027I | Config afterJob の件数ログ4件（ログ種別 PROGRESS）。パラメータは下記 |
| 6 | MBKKSJB0051E | MSGID_MBKKSJB0051E | Writer write の登録エラーログ2件（ログ種別 TABLE_INSERT）。パラメータは下記 |
| 7 | MBKKSJB0058I | MSGID_MBKKSJB0058I | Reader init の検索ログ2件（ログ種別 TABLE_SELECT）。パラメータは下記 |
| 8 | MBKKSJB0060E | MSGID_MBKKSJB0060E | Writer write の更新エラーログ（ログ種別 TABLE_UPDATE）。パラメータは下記 |

### 5-1 MBKKSJB0020E の呼出

KKSJB05P001JobConfig.java 75～78行目:

```java
logWriter(LogKindEnum.CATCH_ON_EXHANDLER,
MsgIdEnum.MSGID_MMBKKSJB0020E.getValue(),
new String[] { CommonConstants.COM_LOG_CONTENT_305 });
```

KKSJB05P001Reader.java 109～112行目（select001／select002 の例外時も同形）:

```java
logWriter(LogKindEnum.TABLE_SELECT,
MsgIdEnum.MSGID_MMBKKSJB0020E.getValue(),
new String[] { CommonConstants.COM_LOG_CONTENT_305 });
```

### 5-2 MBKKSJB0027I の呼出（件数ログ4件、すべて Config afterJob）

データ読込件数（90～93行目）:

```java
logWriter(LogKindEnum.PROGRESS, MsgIdEnum.MSGID_MBKKSJB0027I.getValue(),
new String[] { CommonConstants.COM_LOG_CONTENT_305,
CommonConstants.COM_LOG_CONTENT_33,
String.valueOf(customercopyinfoReadCnt) });
```

メール送付情報登録件数（101～104行目）:

```java
logWriter(LogKindEnum.PROGRESS, MsgIdEnum.MSGID_MBKKSJB0027I.getValue(),
new String[] { CommonConstants.MAILSEND_INFO,
CommonConstants.COM_LOG_CONTENT_106,
String.valueOf(mailsendCreateCnt) });
```

SMS送付情報登録件数（112～115行目）:

```java
logWriter(LogKindEnum.PROGRESS, MsgIdEnum.MSGID_MBKKSJB0027I.getValue(),
new String[] { CommonConstants.COM_LOG_CONTENT_3_45,
CommonConstants.COM_LOG_CONTENT_3_60,
String.valueOf(smssendCreateCnt) });
```

利用者控情報管理更新件数（123～126行目）:

```java
logWriter(LogKindEnum.PROGRESS, MsgIdEnum.MSGID_MBKKSJB0027I.getValue(),
new String[] { CommonConstants.COM_LOG_CONTENT_305,
CommonConstants.CUSTOMERCOPYINFO_UPDATE_CNT,
String.valueOf(customercopyinfoUpdateCnt) });
```

### 5-3 MBKKSJB0051E の呼出（Writer write）

メール送付情報の登録エラー時（97～101行目）:

```java
logWriter(LogKindEnum.TABLE_INSERT,
MsgIdEnum.MSGID_MBKKSJB0051E.getValue(),
new String[] { CommonConstants.MAILSEND_INFO,
mailsendEntity.getMypageId(),
mailsendEntity.getMailTransmitSequence(), tmpErrorCode });
```

SMS送付情報の登録エラー時（133～139行目）。第1パラメータが MAILSEND_INFO ではなく COM_LOG_CONTENT_3_45 で、キー項目は Entity ではなく DTO から取得している点がメール側と非対称である:

```java
logWriter(LogKindEnum.TABLE_INSERT,
MsgIdEnum.MSGID_MBKKSJB0051E.getValue(),
new String[] { CommonConstants.COM_LOG_CONTENT_3_45,
outDto.getSmssendDto().getMypageId(),
outDto.getSmssendDto().getSmsTransmitSequence(),
tmpErrorCode });
```

### 5-4 MBKKSJB0058I の呼出（Reader init）

select001 側（101～104行目）。select002 側は第1パラメータが MYPAG_TMP_HOYU_KEIYAKU、件数が kIYKB477002OutEntityList.size() になる（129～132行目）:

```java
logWriter(LogKindEnum.TABLE_SELECT,
MsgIdEnum.MSGID_MBKKSJB0058I.getValue(),
new String[] { CommonConstants.MYPAG_HOYU_KEIYAKU,
String.valueOf(kIYKB477001OutEntityList.size()) });
```

### 5-5 MBKKSJB0060E の呼出（Writer write、166～168行目）

```java
logWriter(LogKindEnum.TABLE_UPDATE,
MsgIdEnum.MSGID_MBKKSJB0060E.getValue(),
new String[] { CommonConstants.COM_LOG_CONTENT_305, outDto.getCustomercopyinfoDto().getInvariantNo(), tmpErrorCode });
```

## 6. 共通部品・フレームワーク側の定義（11項目）

11項目すべてについて、提供クラスと呼出方法がコードで確定した。部品内部の実装本体（チェック処理の中身、ベースクラスのソース）は6ファイルに含まれないため、その範囲は未判明のままである。

### 6-1 CommonUtils.checkMailAddress

- 提供クラス: jp.or.ja_kyosai.wmp.apl.common.utils.CommonUtils（static メソッド）。
- 呼出（KKSJB05P001Processor.java 65～67行目）。引数は契約者メールアドレス（String）、戻り値は Boolean:

```java
Boolean resultFlg = CommonUtils
.checkMailAddress(inEntity.getHolderMailAdress());
```

- 戻り値の使われ方: true の場合はメール送付情報を作成し（getMailsend）、false の場合は、契約者メールアドレスが定数 PHMF_DUMMY_ADDRESS_FULL と一致しない場合に限り SMS送付情報を作成する（getSmssend）。チェック処理そのものの内容（ダミーアドレス判定を含む内部仕様）はコードからは分からない。

### 6-2 CommonUtils.copyBeanToBean

- 提供クラス: CommonUtils（static メソッド）。形式は copyBeanToBean(コピー元, コピー先) で、戻り値は受けない（文として呼出）。
- 呼出は4件すべて確定した:
- Processor updateCustomercopyinfo（225行目）: `CommonUtils.copyBeanToBean(inEntity, customercopyinfoDto);` — コピー元は入れ子の DTO ではなく引数の InDto そのものである。
- Writer write（83行目）: `CommonUtils.copyBeanToBean(outDto.getMailsendDto(), mailsendEntity);`
- Writer write（119行目）: `CommonUtils.copyBeanToBean(outDto.getSmssendDto(), smssendEntity);`
- Writer write（152行目）: `CommonUtils.copyBeanToBean(outDto.getCustomercopyinfoDto(), updateEntity);`
- 使われ方から分かる範囲: コピー後に個別項目の上書き・追加設定が行われる（Processor ではコピーの後にバッチ更新系とメール作成ステータス、表示フラグを設定する）。同名項目コピーの詳細仕様や型変換の有無は、部品本体がないため確定できない。

### 6-3 ExceptionsFilter

- 提供クラス: jp.or.ja_kyosai.wmp.apl.common.exception.ExceptionsFilter。
- 呼出は Writer の3つの catch 節で同形（95、131、164行目）。static メソッド getSqlErrorCode(e) で、引数は Exception、戻り値は String（エラーコード）で、ログパラメータの末尾に渡される:

```java
tmpErrorCode = ExceptionsFilter.getSqlErrorCode(e);
```

### 6-4 ログ出力の仕組み

- ログ出力は、ベースクラスが提供する **logWriter** メソッドで行う。仕様書では部品名が一切出てこなかったが、実コードでは全ログがこの1形式に統一されている:

```text
logWriter(LogKindEnum.<ログ種別>, MsgIdEnum.<メンバー>.getValue(), new String[] { パラメータ...});
```

- 第1引数は LogKindEnum の定数そのもの、第2引数は MsgIdEnum の getValue()、第3引数は String 配列。呼出箇所とパラメータの実例は5に全件まとめた。

### 6-5 ApplicationException

- 提供クラス: jp.or.ja_kyosai.wmp.apl.common.exception.ApplicationException。
- 使い方は全箇所とも `throw new ApplicationException(e);`（コンストラクタ引数は捕捉した Exception）。呼出箇所は Reader のコンストラクタ（68行目）と init の catch 後（113、141行目）、Writer の3つの catch 後（103、139、170行目）:

```java
catch (Exception e) {
// 5. new ApplicationException(e)をスローする。
throw new ApplicationException(e);
}
```

### 6-6 Job のベースクラス

- 実クラス: jp.or.ja_kyosai.wmp.apl.common.AbstractJobBase。KKSJB05P001JobConfig が継承する（42行目）。
- beforeJob／afterJob は Config 側で実装され、フレームワーク（ジョブのリスナーとして Config 自身を登録する形、runJob の.listener(this)）から呼ばれる。仕様書の開始・終了・異常終了ログ（MBBASEB0001I／0002I／0004E）は Config 内に出力コードがなく、ベースクラス側の実装範囲であることが裏付けられた（5の #1～#3 参照）。
- ベースクラスが提供し、Config が使っているもの: フィールド stepExecution、jobBuilderFactory、stepBuilderFactory、sqlSessionFactory、jobContext、jobExecution、メソッド logWriter、getJobContextForInt、setDoChunkError、isAllowStartIfComplete、getCommitInterval、定数 ALLOW_STARTIF_COMPLETE、CHUNK_COUNT_KEY。
- beforeJob の実装（50～53行目）:

```java
public void beforeJob() {
// 1. setDoChunkError(false)を呼び出す。
setDoChunkError(false);
}
```

### 6-7 Reader のベースクラス

- 実クラス: CustomDbItemReader（6-9 参照）。コンストラクタは仕様書どおり (sqlSessionFactory, jobContext, jobExecution) の3引数で、super.setSqlSessionFactory を呼んだ後、jobContext／jobExecution をフィールドに保持する（KKSJB05P001Reader.java 51～59行目）:

```java
public KKSJB05P001Reader(SqlSessionFactory sqlSessionFactory,
ExecutionContext jobContext, JobExecution jobExecution) {
// 1. SQLセッションファクトリを設定する。
super.setSqlSessionFactory(sqlSessionFactory);

// 2. this.jobContextに引数. コンテクストを設定する。
this.jobContext = jobContext;

// 3. this.jobExecutionに引数. エグゼクションを設定する。
this.jobExecution = jobExecution;
```

- 仕様書の「super.init を呼ぶ」は、実コードではコンストラクタ内の try で自クラスの private メソッド init() を呼ぶ形になっている（62～71行目）。super.init の呼出はない。
- ベースクラスが提供し、Reader が使っているもの: setSqlSessionFactory、beforeChunk(null)（init の先頭、82行目）、setDoReaderError、getMapper、setJobContext、logWriter、フィールド jobContext／jobExecution。

### 6-8 setDoChunkError／setDoReaderError

- setDoChunkError(false): Config の beforeJob で呼ぶ（51行目）。提供は AbstractJobBase。
- setDoReaderError(false): Reader の init で呼ぶ（85行目）。提供は CustomDbItemReader。
- いずれも引数は boolean の false で、修飾なしに呼ぶ（継承したメソッド）。

### 6-9 CustomDbItemReader／CustomDbItemWriter

- パッケージ: jp.or.ja_kyosai.wmp.apl.common.item。
- 継承関係は次のとおり。Reader は型引数に InDto を取る。Writer はクラス自体がジェネリクスで、Config から KKSJB05P001Writer\<KKSJB05P001ProOutDto\> として生成される:

```java
public class KKSJB05P001Reader extends CustomDbItemReader<KKSJB05P001InDto> {
```

```java
public class KKSJB05P001Writer<T> extends CustomDbItemWriter<T> {
```

- 参考: Processor のベースクラスは jp.or.ja_kyosai.wmp.apl.common.AbstractDbItemProcessor\<KKSJB05P001InDto, KKSJB05P001ProOutDto\> で、コンストラクタで super.setSqlSessionFactory(sqlSessionFactory) を呼ぶ（KKSJB05P001Processor.java 24～32行目）。

### 6-10 ジョブコンテクストのアクセス方法

- 設定側はベースクラスの setJobContext(キー, 件数)、取得側は Config の getJobContextForInt(キー) を使う。キーは定数文字列そのものか、Enum の getValue() である。
- 設定箇所は4件: Reader が検索件数を COM_STR_SEARCH_CNT で設定（148～149行目）、Writer が登録・更新件数を TBLID_MAILSEND.getValue()、TBLID_SMSSEND.getValue()、COUNT_CUSTOMERCOPYINFOUPDCNT.getValue() で設定（179～186行目）:

```java
setJobContext(CommonConstants.COM_STR_SEARCH_CNT,
kKSJB05P001InDtoList.size());
```

- 取得箇所は Config afterJob の4件で、すべて getJobContextForInt を使う（86、97～98、108～109、119～120行目）:

```java
int customercopyinfoReadCnt = getJobContextForInt(
CommonConstants.COM_STR_SEARCH_CNT);
```

### 6-11 stepExecution の利用

- Config の afterJob で、ベースクラスのフィールド stepExecution を直接使う。getFailureExceptions() の戻り値は List\<Throwable\> として受ける（62行目）:

```java
List<Throwable> exceptions = stepExecution.getFailureExceptions();
```

- getReadCount() は、異常の原因が SQLSyntaxErrorException の場合の判定条件の中で使う（70～74行目）:

```java
if (ex.getCause() != null
&& ex.getCause() instanceof SQLSyntaxErrorException
&& stepExecution.getReadCount() == 0) {
```

## 7. テーブル定義とシーケンス（5件）

コードで判明したのはアクセス経路のみで、**テーブル列定義とシーケンス定義そのものは6ファイルに含まれず、依然として未判明** である（必要な資料: テーブル定義書、シーケンス定義）。

| # | 対象 | コードで判明したこと | 未判明 |
|---|---|---|---|
| 1 | 利用者控情報管理テーブル | Reader の select001（不変番号に紐づく利用者控の取得）／select002（不変番号に紐づかない利用者控の取得）の抽出対象で、Writer の update001 の更新対象。Entity は KIYKB477001OutEntity／KIYKB477002OutEntity／KIYKB477003InEntity | 列定義（列名・型・桁） |
| 2 | メール送付情報テーブル | Writer の MailsendBtTblDbDao.insert の登録対象。Entity は MailsendEntity | 列定義 |
| 3 | SMS送付情報テーブル | Writer の SmssendBtTblDbDao.insert の登録対象。Entity は SmssendEntity | 列定義 |
| 4 | シーケンス（メール送付シーケンス） | MbComnSeqDbDao.selectSeq003 で採番し、戻り値 String を MailsendDto.setMailTransmitSequence に設定してから Entity にコピーする | シーケンスの定義（名前、採番仕様） |
| 5 | シーケンス（SMS送付シーケンス） | MbComnSeqDbDao.selectSeq006 で採番し、戻り値 String を SmssendDto.setSmsTransmitSequence に設定してから Entity にコピーする | シーケンスの定義（名前、採番仕様） |

## 8. プロジェクト規約（5項目）

### 8-1 パッケージ

import 文から全クラスのパッケージが確定した。

| 対象 | パッケージ |
|---|---|
| Job Config（KKSJB05P001JobConfig） | jp.or.ja_kyosai.wmp.apl.job.config |
| Reader | jp.or.ja_kyosai.wmp.apl.job.reader |
| Processor | jp.or.ja_kyosai.wmp.apl.job.processor |
| Writer | jp.or.ja_kyosai.wmp.apl.job.writer |
| InDto／ProOutDto | jp.or.ja_kyosai.wmp.apl.job.dto |
| 共通DTO（CustomercopyinfoDto／MailsendDto／SmssendDto） | jp.or.ja_kyosai.wmp.apl.job.dto.common |
| DAO（KIYKB477DbDao／MailsendBtTblDbDao／SmssendBtTblDbDao／MbComnSeqDbDao） | jp.or.ja_kyosai.wmp.apl.mybatis.dao |
| Entity（KIYKB477001OutEntity／KIYKB477002OutEntity／KIYKB477003InEntity／MailsendEntity／SmssendEntity） | jp.or.ja_kyosai.wmp.apl.mybatis.entity |
| 定数（CommonConstants／JobInfConstants） | jp.or.ja_kyosai.wmp.apl.common.constants |
| Enum（MsgIdEnum／TableIdEnum／CountEnum） | jp.or.ja_kyosai.wmp.apl.common.constants.enums |
| LogKindEnum | jp.or.ja_kyosai.wmp.apl.common.log |
| 例外（ApplicationException／ExceptionsFilter） | jp.or.ja_kyosai.wmp.apl.common.exception |
| CommonUtils | jp.or.ja_kyosai.wmp.apl.common.utils |
| ベースクラス（AbstractJobBase／AbstractDbItemProcessor） | jp.or.ja_kyosai.wmp.apl.common |
| ベースクラス（CustomDbItemReader／CustomDbItemWriter） | jp.or.ja_kyosai.wmp.apl.common.item |

### 8-2 バージョン

- Java、Spring Batch、MyBatis のバージョン番号は、コードの import からは特定できない。import から分かる使用ライブラリは、Spring Batch（org.springframework.batch 配下）、Spring Framework のアノテーション、MyBatis（org.apache.ibatis.session.SqlSessionFactory）、Apache Commons Lang3（org.apache.commons.lang3.StringUtils、Processor のみ）、Lombok（DTO のみ）である。
- ビルドは Gradle である（DTO の写真のエディタステータスバーに「Gradle: Build Error」の表示があったことからの観察）。ビルド定義ファイル（build.gradle／pom）本体は未送付で、依存ライブラリのバージョンは確定できない。

### 8-3 アノテーション

- Config クラス: @Configuration、@EnableBatchProcessing、@PropertySource（value は "classpath:jobs/config/KKSJB05P001/param_layout.properties"、encoding は CommonConstants.ENCODING_UTF8）（38～42行目）:

```java
@Configuration
@PropertySource(value = {
"classpath:jobs/config/KKSJB05P001/param_layout.properties" }, encoding = CommonConstants.ENCODING_UTF8)
@EnableBatchProcessing
public class KKSJB05P001JobConfig extends AbstractJobBase {
```

- メソッド: @Bean（4メソッド。runJob のみ @Bean(name = JobInfConstants.JobID.KKSJB05P001) と名前指定）、@StepScope（reader と writer の2メソッド）。
- @Override: Reader の read()（295行目）と Writer の write()（50行目）に付く。Processor の process() には付いていない。
- DTO: Lombok の @Getter／@Setter（クラスレベル）。

### 8-4 DTO の getter／setter 規約

- InDto／ProOutDto では **Lombok の @Getter／@Setter**（クラスレベル）で確定した（1-1、1-2 参照）。
- 共通DTO と Entity の定義本体は未送付だが、呼出側で使われるアクセサはすべて標準形式の get〜／set〜 で、フィールド名から機械的に決まる名前になっている（1-3～1-7 の対応表のとおり）。

### 8-5 ジョブの起動・設定

- ジョブ定義は XML ではなく **Java Config** である。Config の runJob がジョブを組み立てる（136～145行目）:

```java
@Bean(name = JobInfConstants.JobID.KKSJB05P001)
public Job runJob(RunIdIncrementer jobParametersIncrementer) {

// ジョブを定義して実行を行う。
return jobBuilderFactory.get(JobInfConstants.JobID.KKSJB05P001)
.incrementer(jobParametersIncrementer).listener(this)
.start(kksjb05p001_S01()).build();
}
```

- ステップ定義（148～159行目）。仕様書のプロパティ job.allowStartIfComplete＝true は isAllowStartIfComplete(ALLOW_STARTIF_COMPLETE) に、job.commitInterval＝1 は getCommitInterval(CHUNK_COUNT_KEY) に、それぞれベースクラスの定数キー経由で対応している。値そのものはプロパティファイル（param_layout.properties）側にあり、そのファイルは未送付である:

```java
@Bean
public Step kksjb05p001_S01() {
return stepBuilderFactory.get(JobInfConstants.StepID.KKSJB05P001_S01)
.listener(this)
.allowStartIfComplete(
isAllowStartIfComplete(ALLOW_STARTIF_COMPLETE))
.<KKSJB05P001InDto, KKSJB05P001ProOutDto>chunk(
this.getCommitInterval(CHUNK_COUNT_KEY))
.reader(this.reader()).processor(this.processor())
.writer(this.writer()).build();
}
```

## 9. 依然として未判明のもの（まとめ）

6ファイルで確定できなかったものは次のとおりである。これらが揃うと、不足情報リストの項目はすべて解消する。

1. SQL 本体と Mapper 定義（select001、select002、update001、MailsendBtTblDbDao.insert、SmssendBtTblDbDao.insert、selectSeq003、selectSeq006 の7メソッド分）
2. DAO 4クラス（KIYKB477DbDao、MailsendBtTblDbDao、SmssendBtTblDbDao、MbComnSeqDbDao）のインターフェース定義本体
3. 共通DTO 3クラス（CustomercopyinfoDto、MailsendDto、SmssendDto）のクラス定義本体（項目の英名は本書で確定したが、型を含む定義そのもの）
4. Entity 5クラス（KIYKB477001OutEntity、KIYKB477002OutEntity、KIYKB477003InEntity、MailsendEntity、SmssendEntity）のクラス定義本体
5. 定数の値（CommonConstants の各定数と、JobInfConstants の本ジョブ以外の定数。JobInfConstants の JobID.KKSJB05P001＝"KKSJB05P001" と StepID.KKSJB05P001_S01＝"KKSJB05P001S01" は写真で確定済み。ログの固定文字は文字列そのものが仕様書で判明済みで、本書で定数名が対応付いた）
6. Enum のコード値（MsgIdEnum、TableIdEnum、CountEnum の getValue() が返す値、LogKindEnum の値）
7. メッセージ文言（MBBASEB0001I、MBBASEB0002I、MBBASEB0004E、MBKKSJB0020E、MBKKSJB0027I、MBKKSJB0051E、MBKKSJB0058I、MBKKSJB0060E の8件の本文とプレースホルダ仕様）
8. テーブル列定義（利用者控情報管理テーブル、メール送付情報テーブル、SMS送付情報テーブルの3件）
9. シーケンス定義（メール送付シーケンス、SMS送付シーケンスの2件）
10. バージョンとビルド定義（Java、Spring Batch、MyBatis のバージョン、build.gradle の依存定義）
11. 共通部品の内部仕様（CommonUtils.checkMailAddress のチェック内容、copyBeanToBean のコピー詳細仕様、ExceptionsFilter のエラーコード取得仕様、ベースクラス AbstractJobBase／CustomDbItemReader／CustomDbItemWriter／AbstractDbItemProcessor の定義本体）
