# chogison

## APIの機能
 -ログイン
 -設定内容保管
  -Userテーブルにカラム追加
 -スケジュール返信
  -JSON形式でレスポンス

## モデル
Facility
  id: integer
  name: string
  has_many :facility_events (foreign_key :facility_id)
FacilityEvent
  facility_id: integer
  event_id: integer
  belongs_to :facility
User
  id: integer
  name: string
  has_many :user_events
UserEvent
  user_id: integer
  event_id: integer
  start_at: datetime
  end_at: datetime
  title: string
  belongs_to :user
  has_many :facilities (through :facility_events)
  has_many :facility_events (foreign_key :event_id)
