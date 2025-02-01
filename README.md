# mrubyc-gem-tmg39931
mruby/c sources for TMG39931

## device
https://jp.seeedstudio.com/Grove-Light-Color-Proximity-Sensor-TMG39931-p-2879.html

### data sheet
https://files.seeedstudio.com/wiki/Grove-Light-Gesture-Color-Proximity_Sensor-TMG39931/res/TMG3993.pdf

## sample code

明るさ (LUX) の検知のみ
```ruby
i2c = I2C.new()
tmg = TMG39931.new(i2c)

if !tmg.init
  puts 'Device not found. Check wiring.'
else
  loop do
    data = tmg.get_rgbc_raw
    puts tmg.get_lux(data)
    sleep 0.1
  end
end
```


```ruby
i2c = I2C.new()
tmg = TMG39931.new(i2c)

if !tmg.init
  puts 'Device not found. Check wiring.'
else
  tmg.setup_recommended_config_for_proximity
  tmg.set_proximity_interrupt_threshold(25, 150) # less than 5cm sill trigger the proximity event
  tmg.set_adc_integration_time(0xdb) # the integration time: 103ms
  tmg.enable_engines(TMG39931::ENABLE[:PON] | TMG39931::ENABLE[:PEN] | TMG39931::ENABLE[:PIEN] | TMG39931::ENABLE[:AEN] | TMG39931::ENABLE[:AIEN])
  last_interrupt_state = -1
  loop do
    if (tmg.get_status & (TMG39931::STATUS[:PINT] | TMG39931::STATUS[:AVALID])) != 0
      proximity_raw = tmg.get_proximity_raw  # read the Proximity data will clear the status bit
      if proximity_raw >= 150 && last_interrupt_state != 1
        puts 'Proximity detected!!!'
        puts "Proximity Raw: #{proximity_raw}"
        data = tmg.get_rgbc_raw
        lux = tmg.get_lux(data)
        cct = tmg.get_cct(data)
        puts '-----'
        puts "RGBC Data: #{data[:r]} #{data[:g]} #{data[:b]} #{data[:c]}"
        puts "Lux: #{lux}   CCT: #{cct}"
        puts '-----'
        last_interrupt_state = 1
      elsif proximity_raw <= 25 && last_interrupt_state != 0
        puts 'Proximity removed!!!'
        puts "Proximity Raw: #{proximity_raw}"
        last_interrupt_state = 0
      end
      # don't forget to clear the interrupt bits
      tmg.clear_proximity_interrupts
    end
    sleep 0.01
  end
end
```
