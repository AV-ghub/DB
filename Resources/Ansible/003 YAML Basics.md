[src](https://www.tutorialspoint.com/ansible/ansible_yaml_basics.htm)   
## key-value pair
```
--- #Optional YAML start syntax 
james: 
   name: james john 
   rollNo: 34 
   div: B 
   sex: male 
… #Optional YAML end syntax 
```
=
```
James: {name: james john, rollNo: 34, div: B, sex: male}
```
## List
```
---
countries:  
   - America 
   - China 
   - Canada 
   - Iceland 
…
```
=
```
Countries: [‘America’, ‘China’, ‘Canada’, ‘Iceland’] 
```

> YAML uses “|” to ***include newlines*** while showing multiple lines and “>” to ***suppress newlines*** while showing multiple lines.
