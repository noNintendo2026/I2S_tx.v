# Protocolo I2S (Inter-IC Sound)
Es un protocolo de bus serie sincronico, que se diseño especificamente para transmitir datos de **audio digital**.

Este protocolo usa principalmente tres lineas de señal **BCLOCK, RLCLOK Y SD** sin embargo tambien puede llegar a usar una cuarta linea de señal **MCLOCK**

---
## BCLOCK

Esta señal son impulsos los cuales permiten sincronisar la informacion y evitar desfaces que dañen el audio, durante la bajada del impulso se carga el bit de informacion que se enviara y en la subida se envia, de esta forma se permite que el bit de informacion se envie de forma correcta.

la frecuencia de esta señal depende de la frecuencia de muestreo, los canales de audio y la profundidad del audio. 

**F_Muestra\*N\*N_canales**
---
## RLCLOCK

En este caso la señal determina a que canal de audio corresponde el bit, su frecuencia es la misma que la de muestreo.

## SD 

Es la informacion del audio **PCM (Modulación por Código de Pulsos)** que se envia bit a bit a un **DAC** el cual se encarga de convertir esta informacion digital a una analoga en forma de sonido.

## MCLOCK

Esta cuarta señal aunque no es nesesaria en todos los casos es la señal con mayor frecuencia siendo dociento cincuenta y seis veces la frecuencia de muestreo y al ser una frecuencia mucho mayor se usa para hacerles modificaciones a la señal de audio

![I2S](Imagenes/i2s.gif)