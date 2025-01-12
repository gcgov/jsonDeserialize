# Json Deserialize Utility

JSON Deserialize is an abstract class that enables JSON deserialization into a specific class. Simply extend the jsonDeserialize class and then call the static jsonDeserialize method. Requires all properties to be typed. Array type will be
determined by a PHPDoc definition.

Requires &gt;=PHP 8

## Installation
`composer require andrewsauder/json-deserialize`

## Usage
Extend your class from `\andrewsauder\jsonDeserialize\jsonDeserialize` class then call ` {yourClass}::jsonDeserialize( {stringified json} );` to deserialize the JSON automatically into your class.

## Attributes
### excludeJsonDeserialize
Tag properties on your class with the `#[excludeJsonDeserialize]` attribute to prevent the value of the field from being **deserialized** into the class generated by `jsonDeserialize()`

### excludeJsonSerialize
Tag properties on your class with the `#[excludeJsonSerialize]` attribute to prevent that field from being **serialized** into the output of `json_encode()`;

### jsonSerializeDateTimeFormat(string $format)
Tag properties on your class that implement DateTimeInterface with the `#[jsonSerializeDateTimeFormat('Y-m-d')]` attribute to force a specific format output. When this attribute is not present, the `DATE_ATOM` format is used by default.

## Hooks
The hooks provided allow you ti extend the default functionality of jsonDeserialize. Add hook methods to your class that extends `\andrewsauder\jsonDeserialize\jsonDeserialize`. The hook will be called automatically during the deserialization and serialization lifecycle.

### Before Json Deserialize
Static method called on a class immediately before deserialization into the class occurs.
```php 
protected static function _beforeJsonDeserialize( string|\stdClass $json ): void {}
```

### After Json Deserialize
Called on a newly created instance after deserialization is complete.
```php 
protected function _afterJsonDeserialize() : void {}
```

### Before Json Serialize
Called on an instance immediately before serialization to JSON.
```php 
protected function _beforeJsonSerialize() : void {}
```

### After Json Serialize
Called on an instance after serialization to a plain array. The standard export data is provided to the hook and the hook must return an array which will be immediately encoded to JSON.
```php 
protected function _afterJsonSerialize( array $export ): array {
    return $export;
}
```

## Code Example

myModel.php

```php
class myModel extends \andrewsauder\jsonDeserialize\jsonDeserialize {

	public int    $varA = 1;
	
	public string $varB = 'B1';
	
	#[excludeJsonDeserialize]
	public string $varC = 'C1';
	
	/** @var string[]  */
	public array $varD = [ 'D1', 'D2', 'D3' ];
	
	public int    $varASquared = 1;
	
	protected function _afterJsonDeserialize() {
		//automagically called after self::jsonDeserialize() has finished its deserialization
		$this->varASquared = $this->varA * $this->varA;
	}
	
	protected function _beforeJsonSerialize() {
		//automagically called before json_encode( {$this} ) serializes object into JSON
		$this->varASquared = $this->varA * $this->varA; 
	}
}
```

myController.php

```php
class myController {
    
    public function post() {
        
        $jsonString = '{ "varA":2, "varASquared":3, "varB":"B2", "varC":"C2", "varD":[ "D4", "D5", "D6" ] }';
        $myModel = myModel::jsonDeserialize( $jsonString );
        
        //$myModel is now an instance of myModel
        echo $myModel->varA;
        echo $myModel->varASquared; //<-Note that this property is updated in _afterJsonDeserialize
        echo $myModel->varB;
        echo $myModel->varC; //<-This property's value is not set from the JSON because it has #[excludeJsonDeserialize]
        foreach( $myModel->varD as $i=>$v) {
                echo $myModel->varD[ $i ];
        }
      
    }
    
}
```

Output:

```
2
4
B2
C1
D4
D5
D6
```

## Debug Logging

To enable debug logging, add these lines prior to deserializing. All objects deserialized after will include debug logging to a new log file in the provided path.
The logging is expensive but it is useful for determining missing properties on classes or json data.

```php
\andrewsauder\jsonDeserialize\config::setDebugLogging( true );
\andrewsauder\jsonDeserialize\config::setDebugLogPath( 'C:/inetpub/logs' );
```

### Fine Tune Logging

By default, when logging is enabled, all debugging messages are enabled.

- To turn off messages about properties that exist in the class but do not exist in the JSON source:
  `\andrewsauder\jsonDeserialize\config::setLogJsonMissingProperty( true ); `

- To turn off messages about properties that exist in the JSON source but do not exist in the class:
  `\andrewsauder\jsonDeserialize\config::setLogClassMissingProperty( true ); `

- To turn off messages about properties in the class without a strict type:
  `\andrewsauder\jsonDeserialize\config::setLogClassPropertyMissingType( true ); `

