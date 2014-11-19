# Kacper Czechowicz - Zadanie 2, agregacje

Dane o ludziach, krajach w których żyją, ich płci, dacie urodzenia, miesięcznych zarobkach oraz ilości posiadanych samochodów wygenerowałem na stronie [http://www.mockaroo.com/](http://www.mockaroo.com/schemas/download).

Przykładowy rekord:
```javascript
{
	"id":1,
	"sex":"Male",
	"first_name":"Craig",
	"last_name":"Parker",
	"country":"Vietnam",
	"salary":778607,
	"birth":"11/13/1972",
	"carsOwned":0
}
```

Ilość danych:
```javascript
> db.people.count();
1000702
```

## Agregacja 1

Ile osób z bazy żyje w danym kraju malejąco:

Javascript:
```javascript
> db.people.group({ 
	key: { country: true},
	initial: {count: 0}, 
	reduce: function(doc, out) {out.count++;} 
}).sort(function(a,b){ return b.count - a.count; });
```
PHP
```php
<?php
	$m = new MongoClient("localhost");
	$collection = $m->selectDB("agregacje")->selectCollection("people");
	
	$keys = array("country" => 1);
	$initial = array("count" => 0);
	$reduce = "function(doc, out) {out.count++;}";
	
	$g = $collection->group($keys, $initial, $reduce);

	$ret = $g['retval'];

	$count = array();
	foreach ($ret as $key => $row)
	{
	    $count[$key] = $row['count'];
	}
	array_multisort($count, SORT_DESC, $ret);

	echo json_encode($ret);
?>
```

Wynik w postaci tabelki: [agregacja1.md](wyniki/agregacja1.md)

## Agregacja 2

Średnia posiadanych samochodów przez kobiety i mężczyzn:

Javascript
```javascript
db.people.group({ 
	key: { sex: true}, 
	initial: {count: 0, total_cars: 0}, 
	reduce: function(doc, out) {out.count++; out.total_cars+=doc.carsOwned;},  
	finalize: function(out) { out.avg_carsOwned = out.total_cars / out.count; }
});
```
PHP
```php
<?php
	$m = new MongoClient("localhost");
	$collection = $m->selectDB("agregacje")->selectCollection("people");
	
	$keys = array("sex" => 1);
	$initial = array("count" => 0, "total_cars" => 0);
	$reduce = "function(doc, out) {out.count++; out.total_cars+=doc.carsOwned;}";
	$finalize = "function(out) { out.avg_carsOwned = out.total_cars / out.count; }";
	$options = array("finalize" => $finalize);

	$g = $collection->group($keys, $initial, $reduce, $options);

	echo json_encode($g['retval']);
?>
```

Wynik: 
![agregacja2](wyniki/agregacja2.png)

Json: [agregacja2.json](wyniki/agregacja2.json)

## Agregacja 3

Suma kobiet i mężczyzn dla każdej ilości samochodów posiadanych:

Javascript
```javascript
db.people.group({ 
	key: { carsOwned: true}, 
	initial: { male: 0, female: 0 }, 
	reduce: function(doc, out) { if(doc.sex === "Male") out.male++; else out.female++; },  
}).sort(function(a,b){ return b.carsOwned - a.carsOwned; });
```
PHP
```php
<?php
	$m = new MongoClient("localhost");
	$collection = $m->selectDB("agregacje")->selectCollection("people");
	
	$keys = array("carsOwned" => 1);
	$initial = array("male" => 0, "female" => 0);
	$reduce = "function(doc, out) { if(doc.sex === 'Male') out.male++; else out.female++; }";
	
	$g = $collection->group($keys, $initial, $reduce);

	$ret = $g['retval'];

	$carsOwned = array();
	foreach ($ret as $key => $row)
	{
	    $carsOwned[$key] = $row['carsOwned'];
	}
	array_multisort($carsOwned, SORT_DESC, $ret);

	echo json_encode($ret);
?>
```

Wynik:
![agregacja3](wyniki/agregacja3.png)

Json: [agregacja3.json](wyniki/agregacja3.json)

## Agregacja 4

5 krajów w ktorych najlepiej zarabiają kobiety:

Javascript
```javascript
db.people.group({ 
	cond: { sex : "Female" },
	key: { country: true}, 
	initial: { count: 0, total_salary: 0 }, 
	reduce: function(doc, out) { out.count++; out.total_salary+= doc.salary; },  
	finalize: function(out) { out.avg_salary = out.total_salary / out.count; }
}).sort(function(a,b){ return b.avg_salary - a.avg_salary; }).slice(0, 5);
```
PHP
```php
<?php
	$m = new MongoClient("localhost");
	$collection = $m->selectDB("agregacje")->selectCollection("people");
	
	$keys = array("country" => 1);
	$initial = array("count" => 0, "total_salary" => 0);
	$reduce = "function(doc, out) { out.count++; out.total_salary+= doc.salary; }";
	$finalize = "function(out) { out.avg_salary = out.total_salary / out.count; }";
	$cond = array("sex" => "Female");
	$options = array("finalize" => $finalize, "condition" => $cond);

	$g = $collection->group($keys, $initial, $reduce, $options);

	$ret = $g['retval'];

	$avg_salary = array();
	foreach ($ret as $key => $row)
	{
	    $avg_salary[$key] = $row['avg_salary'];
	}
	array_multisort($avg_salary, SORT_DESC, $ret);


	echo json_encode(array_slice($ret, 0, 5));
?>
```

Wynik:
![agregacja4](wyniki/agregacja4.png)

Json: [agregacja4.json](wyniki/agregacja4.json)