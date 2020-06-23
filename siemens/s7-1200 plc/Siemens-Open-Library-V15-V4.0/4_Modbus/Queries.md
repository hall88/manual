### Queries 
Any number of queries can be added, the library will take care of executing them, one by one.

Note: mb_query need whole bytes to function, if discrete in or outputs are going to be queried, and the bits dosen't add up to whole bytes, then mb_query_bits should be used.

```pascal
// First query example
#mb_query(mb_addr := 123,
          mode := #mb_query.c.read.holding_reg,
          data_addr := 456,           
          data_ptr := #resault_data_query_1);
//
// A standard modbus telegram
// .---------.--------.---------------.--------------.---  ..  ..  -- --.---------------.
// | mb_addr |  mode  |   data_addr   |   data_len   |      data_ptr    |      CRC      |
// '---------'--------'---------------'--------------'---- --  ..  ..  -'---------------'
//
// --- mb_addr:
// Station address. 1-247 (RTU device address)
//
// --- mode:
// Modbus function code, supported codes: 1, 2, 3, 4, 5, 6, 15 and 16. 
// Keep in mind that modes are shifted, e.g. fc 3 corresponds to mode 103.
// Start typing "#mb_query.c." and the auto complete list will show all 
// available function codes. 
//
// --- data_addr:
// Data address, the actual value that will be sent in the modbus telegram.
// Eg. to read holding address 123, you simply pass 123 to data_addr and
// "#mb_query.c.read.holding_reg" (103) to the mode-parameter. 40001 should 
// NOT be added to 123.
//
// --- data_len:
// Data length. Can be omitted and will then be calculate automatically. 
// However if it's set manualy for one query, then it need to be set back
// to auto for the next query. (#mb_query.c.auto_len)
//
// --- data_ptr:
// Container for the data that will be send or receive. The parameter is 
// not limited to words, but can contain any data types, including: real,
// arrays and udt's. The modbus protocol limit the data of one query to 
// 123 words. (Sometimes up to 125 words)
// IMPORTANT: 
// Bools in a udt or a array has to add up to whole bytes. If another
// number of bools need to be queried, then use the function mb_query_bits.
// This is helpful when reading discrete inputs and outputs.


// Similar query as above, except one input register are read. 
#mb_query(mb_addr := __MB_ADDR__ ,
          mode := #mb_query.c.read.input_reg,
          data_addr := 1234,
          data_ptr := #data_query_2);


// Modicon convention addressing:
#mb_query(mb_addr := __MB_ADDR__ ,
          mode := #mb_query.c.mode.read,
          data_addr := 40005,           
          data_ptr := #data_query_3);
		   
//   - #mb_query.c.mode.read           -> mb function code: 1, 2, 3 and 4  
//   - #mb_query.c.mode.write          -> mb function code: 15 and 16      
//   - #mb_query.c.mode.write_single   -> mb function code: 5 and 6        
// 
// When the Modicon convention is used, the function code is "embedded" in
// to data_addr. Eg. query 40005 will query holding register with address 
// (offset) 4. 

// Some (old) modbus devices doesn't support modbus function code "16" In 
// that case function code number "6" need to be used 
#mb_query(mb_addr := 42,
          mode := #mb_query.c.write.single_holding_reg,
          data_addr := 42,           
          data_ptr := #data_ptr_query_4);
	  

// A query where data_len is set manually. It's possible but not recommaned 
// since it increase the changes of bugs.
#mb_query(mb_addr := 123,
          mode := #mb_query.c.write.holding_reg,
          data_addr := 456,          
          data_len := 2, 
          data_ptr := #data_ptr_query_5); 

// data_len need to be set back to auto, after the query.
#mb_query.data_len := #mb_query.c.auto_len;            


// A query that has a condition for execution.
IF ___conditon_of_exec_query___ THEN
    #mb_query(mb_addr := 123,
              mode := #mb_query.c.write.holding_reg,
              data_addr := 12345,              
              data_ptr := #query_data);
END_IF;
// When then number of queries changes with different plc-scan, 
// like above. Then the library gets a little "hiccup". If the 
// if-statement changes rapidly, then the example can break the
// whole modbus application.	  	  	  
```
### Discreate queries 

mb_query need whole bytes to work, if discrete in or outputs are going to be queried, and the bits dosen't add up to whole bytes, then mb_query_bits should be used.

```pascal
// 7 bits
#mb_query.mb_addr := __MB_ADDR__;
#mb_query.mode := #mb_query.c.read.discrete_input;
#mb_query.data_addr := __DATA_ADDR__;
"mb_query_bits"(data_ptr := array_of_seven_bools, 
                    mb_query := #mb_query);

// One bit
#mb_query.mb_addr := __MB_ADDR__;
#mb_query.mode := #mb_query.c.write.discrete_output;
#mb_query.data_addr := __DATA_ADDR__;
"mb_query_bits"(data_ptr := array_of_one_bool, 
                    mb_query := #mb_query);

// - data_len will be dertmined based on to the length of the array. 
// - The array can only have one dimension.
```

