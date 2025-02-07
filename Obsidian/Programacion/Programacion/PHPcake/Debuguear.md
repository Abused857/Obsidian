devolver formato json de una request para saber lo que envía / recibe PHP cake 3.3

```php
$formData = $this->request->data();

$this->response->type('application/json');

$this->response->body(json_encode($formData, JSON_PRETTY_PRINT));

return $this->response;



 debug($existingCost);

                exit;

``` 

[[Programación]]