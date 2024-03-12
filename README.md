# gkr



### zkVM example code
```rust

let meta = gkr_init(config).add_rom(ROM).add(RAM);

// c = xor_op(a,b)
let xor_op_dag = meta.create_gkr_dag(|meta| {
    let (mem, stack, constant) = util(meta)
    let (a, b, c) = (meta.var(), meta.var(), meta.var())
    let a_bytes = (0..32).map(meta.var())
    let b_bytes = (0..32).map(meta.var())
    let c_bytes = (0..32).map(meta.var())
    // byte_decompose_gate(meta, output, inputs)
    byte_decompose_gate(meta, a, a_bytes)
    byte_decompose_gate(meta, b, b_bytes)
    byte_decompose_gate(meta, c, c_bytes)
    // xor_gate internally doing `logup` accumulation
    (0..32).for_each(|_| xor_gate(meta, &c_bytes[i], &a_bytes[i], &b_bytes[i]))
    stack.pop(a)
    stack.pop(b)
    stack.push(c)
})
let xor_op_gkr = convert_to_layers_gkr(xor_op_dag)
let data_par = xor_op_data_par(xor_op_gkr, 1 << 10) // 2^10 max instances for asymmetric symcheck

// zero_op(c) => c = 0
meta.create_gkr_dag(|meta| {
    let (_, stack, _) = util(meta)
    let zero = meta.const(0)
    let c = meta.var()
    stack.pop(c)
    // assert c equal to 0 via LogUp on constant ROM table
    eq_gate(c, zero)
})

// chain from output to input
let gkr_circuit = empty_layers.chain(
    output_layer
).chain( // binary tree aggregation
    tree_structure_layer
).chain(
    // data parallel
    data_par_layers(
        [
            xor_op_gkr,
            ... // more opcode
        ]
    )
)

let proofs = create_proof(gkr_circuit, instance)
let _ = verify_proof(gkr_circuit, proofs)

```