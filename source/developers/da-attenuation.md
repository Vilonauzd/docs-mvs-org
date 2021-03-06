title: MST Conditional Offering
---

# Concept
Attenuation model is an advanced lock mechanism. This model will lock some specified quantity of assets, and unlock them gradually in a specified way(specified time interval and quantity).

In fact, the Digital Aseets Lock-Unlock model is the function combinated by MIP-3(Digital Assets Secondary Issue), MIP-7 Digital Aseets - Lock), and MIP-8(Digital Aseets - Unlock). We decide to list a series of adjustable parameters for user to adjust by themselves, like Issue/issuance quantitiy, Freeze quantity, Release period, etc.

ref <https://github.com/mvs-org/mips/issues/7> for more information.

# Usage
* How to specify attenuation model param?
    The param is specified in `issue` and `secondaryissue` and `[did]sendasset` and `[did]sendassetfrom` command, through `-m [--model]` option.
* How to know the locked asset quantity?
    Through `getaccountasset` and `getaddressasset` command, is the value corresponding to key **"locked_quantity"**.
* How to know the attenuation model param?
    Through `gettx` and `lsittxs` comand, is the json object corresponding to key **"attenuation_model_param"**.
* How is the locked asset be unlocked?
    The locked asset is unlocked according to the block height. No need to manually unlock them, and it can be spend directly if the locked time interval is passed. And the unlocked quantity is also calulated automatically.

# Supported Model Type
1. fixed quantity ( TYPE = 1 )
> This model will unlock the assets with equal quantity and with equal time interval.
> The last cycle may have some difference as remainder of division.

2. custom ( TYPE = 2 )
> This model will unlock the assets with quantity and time interval customized by user.

3. fixed inflation ( TYPE = 3 )
> This model unlock Token with a Inflation rate release model for each period and dynamically adjust the Quantity of unlocks per period.

# Model Param's Format
1. = (equal sign) separates key and value of key-value entry.
2. ; (semicolon) separates key-value entries.
3. , (comma) separates items of value in key-value entry.
4. The keys of each model are restricted to given ones, no more and no less.
    For `fixed quantity model`, the keys are `PN, LH, TYPE, LQ, LP, UN`
    For `custom model`, the keys are `PN, LH, TYPE, LQ, LP, UN, UC, UQ`
    For `custom model`, the keys are `PN, LH, TYPE, LQ, LP, UN, IR`
    Among these keys, `PN, LH` are mutable and generated by program(user should not imput these two keys).
    The others are immutable, after inited, they will never change any more.
5. The keys' meaning
    ```
    In order to facilitate the description of subsequent calculation formulas,
    we define the following parameters in a unified way：

    IQ：Total quantity of utxo at this time, must >0
    LQ：Total lock quantity at this time, must >=0
    LP：Total lock period, in block height units, must >0 and be the integer number
    UC_t：Unlock period, in block height units, defines the unlock interval.
    UQ_t: Quantity of Unlocks in period t
    IR_t: Inflation Rate in period t

    concrete names for short keys:

    PN      current_period_nbr
    LH      next_interval
    TYPE    type
    LQ      lock_quantity
    LP      lock_period
    UN      total_period_nbr
    IR      inflation_rate
    UC      custom_lock_number_array
    UQ      custom_lock_quantity_array
    ```

# Constraints values must satisfy
* for fixed quantity model
```
    TYPE=1,
    UN>0,
    LQ<=IQ(utxo asset value)
    LQ>=UN
    LP>=UN
```
* for custom model
```
    TYPE=2,
    UN>0, UN<=100
    LQ<=IQ(utxo asset value)
    both UC and UQ array size are equal to UN, and with positive item.
    LQ = sum of UQ array items
    LP = sum of UC array items
```
* for fixed inflation model
```
    TYPE=3,
    UN>0, UN<=100
    LQ=IQ(utxo asset value)
    LQ>=UN
    LP>=UN
    IR>0, IR<=100000
```

# Examples
* example of fixed quantity model param
    **user input: "TYPE=1;LQ=9001;LP=60001;UN=3"**
    **after init: "PN=0;LH=20000;TYPE=1;LQ=9001;LP=60001;UN=3"**
```
    This model means 9001 (LQ) assets will be locked in 60001 (LP) block heights.
    These assets will be unlocked in 3 (UN) periods with fixed quantity and interval.
    We can infer that **UC=LP/UN** and **UQ=LQ/UN** (the last unlock all left)
    That is  UC=60001/3=20000 and UQ=9001/3=3000, so,
    * after 20000 block height, 3000 will be unlocked,
    * after another 20000 block height, another 3000 will be unlocked,
    * after the left 20001 block height, the all lefted 3001 will be unlocked.
```
* example of custom model param
    **user input: "TYPE=2;LQ=9001;LP=60001;UN=3;UC=20000,20000,20001;UQ=3000,3000,3001"**
    **after init: "PN=0;LH=20000;TYPE=2;LQ=9001;LP=60001;UN=3;UC=20000,20000,20001;UQ=3000,3000,3001"**
```
    This custom model has the same effect with the above example.
    It's unlock quantity and time interval is specified by arrays.
```
* example of fixed inflation model param
    **user input: "TYPE=3;LQ=20000000;LP=12000;UN=12;IR=8"**
    **after init: "PN=0;LH=1000;TYPE=3;LQ=20000000;LP=12000;UN=12;IR=8;UC=...;UQ=..."**
```
    This model will calculate UC and UQ in the initialization,
    and then use it just like a cumstom model from then on.
```

