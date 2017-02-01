# Modbus TCP protocol client

* Modbus TCP/IP specification: http://www.modbus.org/specs.php
* Modbus TCP/IP simpler description: http://www.simplymodbus.ca/TCP.htm

##Supported functions

* FC1 - Read Coils
* FC2 - Read Input Discretes
* FC3 - Read Holding Registers
* FC4 - Read Input Registers
* FC5 - Write Single Coil
* FC6 - Write Single Register
* FC15 - Write Multiple Coils
* FC16 - Write Multiple Registers

## Requirements

* PHP 5.6+

## Intention
This library is influenced by [phpmodbus](https://github.com/adduc/phpmodbus) library and meant to be provide decoupled Modbus protocol (request/response packets) and networking related features so you could build modbus client with our own choice of networking code (ext_sockets/streams/Reactphp asynchronous streams) or use library provided networking classes (php Streams)

## Example (fc3 - read holding registers)

Some of the Modbus function examples are in `examples/` folder

```php
use ModbusTcpClient\Network\BinaryStreamConnection;
use ModbusTcpClient\Packet\ErrorResponse;
use ModbusTcpClient\Packet\ModbusFunction\ReadHoldingRegistersRequest;
use ModbusTcpClient\Packet\ResponseFactory;

$connection = BinaryStreamConnection::getBuilder()
    ->setHost('192.168.0.1')
    ->build();
    
$startAddress = 12288;
$quantity = 6;
$packet = new ReadHoldingRegistersRequest($startAddress, $quantity); //create FC3 request packet

try {
    $binaryData = $connection->connect()->sendAndReceive($packet);

    //parse binary data to response object
    $response = ResponseFactory::parseResponse($binaryData);
    
    //check if response contains modbus error or use ::parseResponseOrThrow() to throw exception on modbus error packets
    if ($response instanceof ErrorResponse) { 
        throw new \Exception($response->getErrorMessage(), $response->getErrorCode());
    }
    
    echo 'Data parsed from packet (bytes):' . PHP_EOL;
    print_r($response->getData()); // array of bytes. These are not modbus WORDs. 1 WORD is 2 bytes
    print_r($response->getWords()); // array of words. Word in this case array of 2 bytes. [[0,100],[0,22]]

} catch (Exception $exception) {
    echo $exception->getMessage() . PHP_EOL;
} finally {
    $connection->close();
}
```