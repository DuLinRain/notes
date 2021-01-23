# Go Marshal和UnMarshal性能测试


    (base) c02ym00wjg5h:encodejson linyu$ go test ./... -bench=BenchmarkJSONMarshal -benchmem -run=^$ -count=10 > head.txt && benchstat head.txt
    name            time/op
    JSONMarshal-12  290ns ±13%
    
    name            alloc/op
    JSONMarshal-12  48.0B ± 0%
    
    name            allocs/op
    JSONMarshal-12   1.00 ± 0%


    (base) c02ym00wjg5h:encodejson linyu$ go test ./... -bench=BenchmarkJSONUnMarshal -benchmem -run=^$ -count=10 > head.txt && benchstat head.txt
    name              time/op
    JSONUnMarshal-12  337ns ± 2%
    
    name              alloc/op
    JSONUnMarshal-12   352B ± 0%
    
    name              allocs/op
    JSONUnMarshal-12   8.00 ± 0%
