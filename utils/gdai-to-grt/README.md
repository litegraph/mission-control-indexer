# GDAI to GRT Converter

## Install dependencies

```
yarn
```

## Run

```sh
$ ./gdai-to-grt.ts
GRT/GDAI: 10.0
GRT/GDAI (wei): 10000000000000000000

$ ../gdai-to-grt.ts --agora
{"GDAI":"10.0"}
```
or
```
$ yarn gdai-to-grt
GRT/GDAI: 10.0
GRT/GDAI (wei): 10000000000000000000

$ yarn agora
{"GDAI":"10.0"}
```

## Why?

This allows you to convert easily from GDAI to GRT in your cost models. 
For instance, the following rule prices all queries as 2 GDAI if there were a 
variable called `$GDAI`:

```
default => 2 * $GDAI
```
