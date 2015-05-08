# create-sql

SQL Generator for node.js, fork of [this package](https://github.com/Shanon/node-sql-generator)

## Installation

    npm install create-sql

## Examples

### INSERT

```js
var sqlgen = require('create-sql');
    
var stmt = sqlgen.insert('test_table', {
  foo: 1,
  bar: 'text',
  buz: '2011-10-10',
  ary_txt: ['a', 'b'],
  ary_num: [1, 2]
});

// it return this
// stmt = {
//   sql: 'INSERT INTO test_table ( foo, bar, buz ) VALUES ( $1, $2, $3, ARRAY[ $4, $5 ], $6 )',
//   values: [ 1, 'text', '2011-10-10', 'a', 'a' ], '{1,2}'
// };
```


### SELECT

```js
var sqlgen = require('create-sql');
var stmt = sqlgen.select('base_table', ['id'], {
  id: { '>=': 33 } // where selection
});

// it return this
// stmt = {
//   sql: 'SELECT id FROM base_table WHERE id >= $1',
//   values: [33]
// };


var stmt2 = sqlgen.select('test_table', '*', {
  foo: 1,                                         // foo = 1
  bar: { '>=': 10 },                              // bar >= 10
  buz: { '>': 100, '<': 200 },                    // buz > 100 AND buz < 200
  hoge: { like: '%john%' },                       // hoge LIKE '%john%'
  fuga: { IN: [ 1, 2, 3 ] },                      // fuga IN ( 1, 2, 3 )
  moge: [ 6, 7, 8, { '!=': 9 } ],                 // ( moge = 6 OR moge = 7 OR moge = 8 OR moge != 9 )
  puga: { '-and': [ 1, 2, 3 ] },                  // ( puga = 1 AND puga = 2 AND puga = 3 ),
  '-or': { red: 1, blue: 2, green: { '!=': 3 } }, // ( red = 1 OR blue = 2 OR green != 3 )
  base_table_id: { IN: { sql: stmt } }            // where section
}, { order: 'id' });
    
// it return this
// stmt2 = {
//   sql: 'SELECT * FROM test_table \
//     WHERE foo = $1 AND bar >= $2 AND buz > $3 AND buz < $4 AND hoge LIKE $5 \
//       AND fuga IN ( $6, $7, $8 ) AND ( moge = $9 OR moge = $10 OR moge = $11 OR moge != $12 ) \
//       AND ( red = $13 OR blue = $14 OR green != $15 ) \
//       AND base_table_id IN ( SELECT id FROM base_table WHERE id >= $16 ) ORDER BY id',
//   values: [1, 10, 100, 200, '%john%', 1, 2, 3, 6, 7, 8, 9, 1, 2, 3, 33]
// };

    // columns of type ARRAY
    var ary_stmt = sqlgen.select( 'array_table', // target table
                                  '*',           // target columns
                                  { text_ary1: { ARRAY: 'hoge' },                // 'hoge' = ANY( text_ary1 )
                                    text_ary2: { ARRAY: { '= ALL': 'fuga' } },   // 'fuga' = ALL( text_ary2 )
                                    num_ary1: { ARRAY: [ 1, 2 ] },               // ( 1 = ANY( num_ary1 ) OR 2 = ANY( num_ary1 ) )
                                    num_ary2: { ARRAY: { '!= ALL': [ 1, 2 ] } }, // ( 1 != ALL( num_ary2 ) OR 2 != ALL( num_ary2 ) )
                                    num_ary3: { ARRAY: { sql: stmt } }           // ( SELECT id FR..... ) = ANY( num_ary3 )
                                  },
                                  { order: 'id' } );
    // it return this
    // ary_stmt = { sql: 'SELECT * FROM array_table \
    //                    WHERE $1 = ANY(text_ary1) AND $2 = ALL(text_ary2) AND ($3 = ANY(num_ary1) OR $4 = ANY(num_ary1)) \
    //                          AND ($5 != ALL(num_ary2) OR $6 != ALL(num_ary2)) \
    //                          AND (SELECT id FROM base_table WHERE id >= $7) = ANY(num_ary3) ORDER BY id',
    //              values: [ 'hoge', 'fuga', 1, 2, 1, 2, 33 ] }
```
### UPDATE
```js
    var sqlgen = require('create-sql');
    var stmt = sqlgen.update( 'test_table', // target table
                              { id: 10 }, // where section
                              { foo: 20, bar: 30, buz: 40,
                                ary_num: [ 1, 2 ], ary_txt: ['a','b'] } // update datas
                            );
    // it return this
    // stmt = { sql: 'UPDATE test_table SET foo = $1, bar = $2, buz = $3, ary_num = $4, ary_txt = ARRAY[ $5, %6 ] WHERE id = $7',
                values: [ 20, 30, 40, '{1,2}', 'a', 'b', 10 ] };

    // or callback pattern
    // sqlgen.update( 'table_name', { wheres..... }, { datas..... },
    //                function( sql, values ) {
    //                    ..
    //                    ..
    //                    ..
    //                } );
```
### DELETE
```js
    var sqlgen = require('create-sql');
    var stmt = sqlgen.delete( 'test_table', // target table
                              { some_flag: 1 } // where section
                            );
    // it return this
    // stmt = { sql: 'DELETE FROM test_table WHERE some_flag = $1',
                values: [ 1 ] };
```

## License

[The MIT License](license.md)
