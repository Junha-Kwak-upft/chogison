moongift/ncmb-ruby-client
https://mb.api.cloud.nifty.com/push

require 'ncmb'
require 'date' # TODO delete

class Tasks::Push
  # TODO lib/ncmb/device.rbのクラス名がDeviseである。バグか確認
  
  # TODO キーの管理
  NCMB.initialize application_key: '4266b31e298782609c702122ac53e525ad50802804036a955674d4f7e6329090', client_key: '52d1cd4da6100aee638f30299bb16e1fbbc603f40de0e90d419b0b9427f715b7'

  def self.execute
    # スケジュールを取得する
    from = Time.zone.now
    to = from + 5.minute
    from = DateTime.new(2015,12,6,6,1,1) # TODO delete
    to = DateTime.new(2015,12,6,22,1,1) # TODO delete

    # TODO 検索条件の時間の動作確認
    events = UserEvent.where("user_events.start_at BETWEEN ? AND ?", from.strftime("%Y-%m-%d %H:%M"), to.strftime("%Y-%m-%d %H:%M"))
                      .order("user_events.user_id, user_events.start_at")
                      .includes(:push_manage)

    # 出力文字列サンプル
    # "予定：会議1、開始時間：2015-12-05 10:00、終了時間：2015-12-05 11:00、設備：notebook1 mike "
    # "予定：会議2、開始時間：2015-12-05 17:30、終了時間：2015-12-05 19:00、設備："

    events.each do |user_event|
# binding.pry # TODO delete
      if user_event.push_manage
        next if user_event.push_manage.pushed
      end
  
      # 通知文字列を編集する
      msg = '予定：'
      msg << user_event.title
      msg << '、開始時間：'
      msg << user_event.start_at.strftime("%Y-%m-%d %H:%M") # TODO 時間表示確認
      msg << '、終了時間：'
      msg << user_event.end_at.strftime("%Y-%m-%d %H:%M")

      facilities = Facility.joins("JOIN facility_events ON facility_events.facility_id = facilities.id ")
                           .where("facility_events.event_id = ?", user_event.event_id)
      if facilities
        msg << '、設備：' # TODO 設備ない場合
        facilities.each do |facility|
          msg << facility.name
          msg << ' '
        end
      end

      # プッシュ通知を登録する
      user_tokens = UserToken.where("user_tokens.user_id = ?", user_event.user_id)
      if user_tokens
        pushed = true
        user_tokens.each do |user_token|
          @push = NCMB::Push.new
          @push.immediateDeliveryFlag = true
          # @push.target = ['ios']
          @push.searchCondition = user_token.token
          @push.message = msg
          p msg # TODO delete
          @push.deliveryExpirationTime = "3 day"
          if @push.save
            puts "Push save successful."
          else
            puts "Push save faild."
            pushed = false
          end
        end

        # 通知フラグを更新する
        unless user_event.push_manage
          push_manage = PushManage.new({user_id: user_event.user_id, event_id: user_event.event_id, pushed: pushed})
          push_manage.save
        else
          user_event.push_manage.pushed = pushed
          user_event.push_manage.save
        end
      end
    end
  end
end
