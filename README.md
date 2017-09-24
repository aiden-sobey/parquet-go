# parquet-go v0.3
parquet-go is a pure-go implementation of reading and writing the parquet format file. 
* Support Read/Write Nested/Flat Parquet File
* Support all Types in Parquet
* Very simple to use (like json marshal/unmarshal)

## Required
* git.apache.org/thrift.git/lib/go/thrift
* github.com/golang/snappy

## Types
There are two Types in Parquet: Base Type and Logical Type
They are defined in ParquetType.go as following:
```
//base type
type BOOLEAN bool
type INT32 int32
type INT64 int64
type INT96 string // length=96
type FLOAT float32
type DOUBLE float64
type BYTE_ARRAY string
type FIXED_LEN_BYTE_ARRAY string

//logical type
type UTF8 string
type INT_8 int32
type INT_16 int32
type INT_32 int32
type INT_64 int64
type UINT_8 uint32
type UINT_16 uint32
type UINT_32 uint32
type UINT_64 uint64
type DATE int32
type TIME_MILLIS int32
type TIME_MICROS int64
type TIMESTAMP_MILLIS int64
type TIMESTAMP_MICROS int64
type INTERVAL string // length=12
type DECIMAL string

```
The variables which will read/write from/to a parquet file mush be declare as these types.

## Core Data Structure
The core data structure named "Table":
```
type Table struct {
	Repetition_Type    parquet.FieldRepetitionType
	Type               parquet.Type
	Path               []string
	MaxDefinitionLevel int32
	MaxRepetitionLevel int32

	Values           []interface{}
	DefinitionLevels []int32
	RepetitionLevels []int32
}
```
Values is the column data; RepetitionLevels is the repetition levels of the values; DefinitionLevels is the definition levels of the values.
The architecture of the data struct is following:
Table -> Page
Pages -> Chunk
Chunks -> RowGroup
RowGroups -> ParquetFile

## Marshal/Unmarshal
Marshal/Unmarshal functions are used to encode/decode the parquet file. 
Marshl convert a struct slice to a ```*map[string]*Table```
Unmarshal convert a ```*map[string]*Table``` to a struct slice

The example of Marshal/Unmarshl can be found the Marshal/Unmarshal_test.go

## Read/Write

### Read Example
```
func Read(fname string) {
	file, _ := os.Open(fname)
	defer file.Close()

	res := ReadParquet(file)
	for _, rowGroup := range res {
		for _, chunk := range rowGroup.Chunks {
			for _, page := range chunk.Pages {
				fmt.Println(page.DataTable.Path)
				for i := 0; i < len(page.DataTable.Values); i++ {
					if page.Header.GetType() == parquet.PageType_DATA_PAGE {
						fmt.Println(page.DataTable.Values[i],
							page.DataTable.RepetitionLevels[i],
							page.DataTable.DefinitionLevels[i])
					}
				}
			}
		}
	}
}
```

### Write Example
```
type Student struct{
......
}

stus := make([]Student,10000)
schemaHandler := NewSchemaHandlerFromStruct(new(Student))
......

file, _ := os.Create("flat.parquet")
WriteParquet(file, stus, schemaHandler)

```

## Note
* Have tested the parquet file written by parquet-go on many big data platform (Spark/Hive/Presto), everything is ok :)
* Almost all the features of the parquet are provided now.

## To do
* Parallel
* Optimize performance