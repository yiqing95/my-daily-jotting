根据文件生成SMD 格式的服务描述
可以藉此动态生成JSONRpc 代理

相关链接：
    [zend SMD](https://github.com/zendframework/zf2/blob/master/library/Zend/Json/Server/Smd.php)
    [代码下载](http://the-stickman.com/web-development/generate-php-simple-method-description-smd-files-for-json-rpc-in-dojo/)
~~~
<?php

/**
 * Generates SMD JSON strings for Dojo from PHP 5 class files
 *
 * Can handle multiple classes in a single file
 *
 * Usage:
 *  PHP5SMDGenerator::parseFile( 'some_php5_class_file.php' )
 */
class PHP5SMDGenerator{

	// Main wrapper template
	const template_main = '{ "serviceType": "JSON-RPC", "serviceURL": "[[filename]]", "methods":[ [[methods]] ] }';

	// Method template
	const template_method = '{ "name": "[[name]]", "parameters":[ [[parameters]] ] }';

	// Parameter template
	const template_parameter = '{"name": "[[name]]"}';

	/**
	 * Reads a list of classes in a PHP5 file and outputs a JSON Simple Method description (SMD) file as a string
	 *
	 * @param		string		$filename		Name of the file to parse
	 *
	 * @return		string						SMD file as string
	 */
	public static function parseFile( $filename ){

		// Get list of classes in this file
		$before = get_declared_classes();
		include( $filename );
		$classes_in_file = array_diff( get_declared_classes(), $before );

		// Stores method list
		$methods = array();
		// There may be more than one class in any given file
		foreach( $classes_in_file as $class ){
			// Reflection object for this class
			$class_reflect = new ReflectionClass( $class );

			// Get a list of methods in this class
			foreach ( $class_reflect->getMethods() as $method ){
				// Only list public methods
				if( $method->isPublic() ){
					// Get a list of parameters for this method
					$parameters = array();
					foreach( $method->getParameters() as $parameter ){
						// Parse into parameter template
						$parameters[] = str_replace(
							'[[name]]',
							$parameter->getName(),
							self::template_parameter
						);
					}
					// Parse into method template
					$methods[] = str_replace(
						array(
							'[[name]]',
							'[[parameters]]'
						),
						array(
							// Combine class name and method name
							$class_reflect->getName() . '.' . $method->getName(),
							implode( ',', $parameters )
						),
						self::template_method
					);
				}
			}
		}

		// Parse methods into main list template
		return str_replace(
			array(
				'[[filename]]',
				'[[methods]]'
			),
			array(
				$filename,
				implode( ',', $methods )
			),
			self::template_main
		);

	}
}

?>

~~~