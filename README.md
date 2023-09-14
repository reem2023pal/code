# code
This code for stage one.

from WeServo import *
import lcd
import sensor
from fpioa_manager import fm
from Maix import GPIO
from WePort import *
from WeUltrasonicSensor import *
from WeDCMotor import *
from Maix import I2S
import audio

_img = 0	#img
_angle = 0	#angle

servo_1 = WeServo(1, 0)
fm.register(32, fm.fpioa.GPIOHS2)
pin6 = GPIO(GPIO.GPIOHS2, GPIO.IN)
ultrasonic_A = WeUltrasonicSensor(PORT_A)
dc_1 = WeDCMotor(1)
ultrasonic_B = WeUltrasonicSensor(PORT_B)
ultrasonic_C = WeUltrasonicSensor(PORT_C)
ultrasonic_D = WeUltrasonicSensor(PORT_D)
fm.register(32, fm.fpioa.GPIO1)
voice_en = GPIO(GPIO.GPIO1, GPIO.OUT)
fm.register(33, fm.fpioa.I2S0_WS)
fm.register(34, fm.fpioa.I2S0_OUT_D1)
fm.register(35, fm.fpioa.I2S0_SCLK)
i2s = I2S(I2S.DEVICE_0)
i2s.channel_config(i2s.CHANNEL_1, I2S.TRANSMITTER, resolution=I2S.RESOLUTION_16_BIT, cycles=I2S.SCLK_CYCLES_32, align_mode=I2S.RIGHT_JUSTIFYING_MODE)

servo_1.write_angle(90)
lcd.init(freq=15000000,color=0x0000)
sensor.reset()
sensor.set_pixformat(sensor.RGB565)
sensor.set_framesize(sensor.QVGA)
sensor.run(1)
sensor.skip_frames(10)
if pin6.value() == 0:
	for i in range(439):
		if ultrasonic_A.distanceCM() < 15:
			while not ultrasonic_A.distanceCM() > 50:
				if _angle < 88:
					servo_1.write_angle(104)
				if _angle > 88:
					servo_1.write_angle(68)
				ultrasonic_A.rgbShow(3, 231, 36, 36)
				dc_1.run(-100)
			_img = sensor.snapshot()
			lcd.display(_img)
		else:
			if ultrasonic_B.distanceCM() < ultrasonic_C.distanceCM():
				_angle = 104
				servo_1.write_angle(_angle)
				dc_1.run(230)
				ultrasonic_B.rgbShow(3, 247, 3, 3)
				ultrasonic_C.rgbShow(3, 65, 193, 68)
				ultrasonic_A.rgbShow(3, 65, 193, 68)
			if ultrasonic_C.distanceCM() < ultrasonic_B.distanceCM():
				_angle = 67
				servo_1.write_angle(_angle)
				dc_1.run(230)
				ultrasonic_C.rgbShow(3, 247, 3, 3)
				ultrasonic_B.rgbShow(3, 65, 193, 68)
				ultrasonic_A.rgbShow(3, 65, 193, 68)
			if ultrasonic_D.distanceCM() == ultrasonic_B.distanceCM():
				servo_1.write_angle(88)
				dc_1.run(230)
				ultrasonic_D.rgbShow(3, 72, 202, 2)
				ultrasonic_B.rgbShow(3, 65, 193, 68)
				ultrasonic_A.rgbShow(3, 65, 193, 68)
			_img = sensor.snapshot()
			lcd.display(_img)
	dc_1.run(0)
	servo_1.write_angle(88)
	voice_en.value(1)
	myAudio = audio.Audio(path="/sd/1.wav")
	i2s.set_sample_rate(myAudio.play_process(i2s)[1])
	myAudio.volume(90)
	while not myAudio.play() == 0: pass
	myAudio.finish()
	voice_en.value(0)
