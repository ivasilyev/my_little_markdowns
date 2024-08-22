# Windows registry short reference

## Absolute path for `reg`

```shell script
C:\Windows\System32\reg.exe
```

## Operations (using environment variables as an example)

### Read an entire folder

```shell script
reg query HKEY_CURRENT_USER\Environment
```

### (Over)write the `test_1` value under the `Test1` key and with the `REG_EXPAND_SZ` datatype

```shell script
reg add HKEY_CURRENT_USER\Environment /v Test1 /t REG_EXPAND_SZ /d test_1 /f
```

### Delete the `Test1` key

```shell script
reg delete HKEY_CURRENT_USER\Environment /v Test1 /f
```
