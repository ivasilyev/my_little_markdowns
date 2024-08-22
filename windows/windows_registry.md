# Windows registry short reference

## Absolute path for `reg`

```shell
C:\Windows\System32\reg.exe
```

## Operations (using environment variables as an example)

### Read an entire folder

```shell
reg query HKEY_CURRENT_USER\Environment
```

### (Over)write the `test_1` value under the `Test1` key
```shell
reg add HKEY_CURRENT_USER\Environment /v Test1 /t REG_EXPAND_SZ /d test_1 /f
```

### Delete the `Test1` key

```shell
reg delete HKEY_CURRENT_USER\Environment /v Test1 /f
```
