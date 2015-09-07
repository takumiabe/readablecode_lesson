リーダブルコード勉強会

# 12章 コードに思いを込める

1つのコード例について作業する。

## ロジックを簡潔な言葉で説明する

例にあげたコードを、簡潔な言葉でSTEP分けしてみよう

```rb
def self.inquire
  printing_orders = PrintingOrder.not_finished.all
  state_types = StateType.all.pluck(:id, :status_code)

  printing_order_numbers = printing_orders.pluck(:order_number)
  xml = Nokogiri::XML::Builder.new(encoding: 'UTF-8') do |xmlbuilder|
    xmlbuilder.Orders do
      printing_order_numbers.each do |n|
        xmlbuilder.OrderNo n
      end
    end
  end

  params = {
    check_sum: Digest::MD5.hexdigest(xml),
    inquire_file: UploadIO.new(StringIO.new(xml), 'text/xml', 'order_inquire.xml'),
  }
  response = post(self.inquiry_url, params, true)

  response.xpath('/File/OrderInquiry/OrderStatus').each do |order_status|
    order = printing_orders.detect {|po| po.order_number == order_status['no'] }
    state_type = state_types.detect{|st| st.status_code == order_status.text}

    if order.current_printing_order_state_id == nil or order.current_printing_order_state_id != state_type.id
      self.transaction do
        order.change_state(state_type)
        order.tracking_number = order_status['tracking-no']
        if order.save! and order.completed?
          ParentMailer.delay.shipped(order.id)
        end
      end
    end
  end
end
```

## 複雑な部分を切り分けて関数にする

STEP分けにしたがって、複雑だったり、粒度が違う部分を関数化してみよう

## 切り分けた関数も綺麗にする

切り分けた関数を、元のコンテキストから切り離して整理してみよう
