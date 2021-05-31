# решения задачек с CodeWars

### Disclaimer
please pay no attention to syntax, this isn't how I code at work.
in fact, I don't do syntax at work, linter does 

## 2 kyu Assembler interpreter (part II)
https://www.codewars.com/kata/58e61f3d8ff24f774400002c/
```js
var assemblerInterpreter = (input, prog = input.split('\n')) => {
  var msg = '', mem = {}, cur = 0, cstack = [], lables = {}, cmp = [], finish
    , ops = {
        mov: (reg, val) => mem[reg] = Number.isInteger(mem[val]) ? mem[val] : +val
      , inc: reg => mem[reg]++
      , dec: reg => mem[reg]--
      , add: (reg, val) => mem[reg] += Number.isInteger(mem[val]) ? mem[val] : +val
      , sub: (reg, val) => mem[reg] -= Number.isInteger(mem[val]) ? mem[val] : +val
      , mul: (reg, val) => mem[reg] *= Number.isInteger(mem[val]) ? mem[val] : +val
      , div: (reg, val) => mem[reg] = parseInt(mem[reg] / (Number.isInteger(mem[val]) ? mem[val] : +val))
      , call: lable => {cstack.push(cur); cur = lables[lable]}
      , msg: (...arg) => msg += arg.map(el => el[0] == '\'' ? el.slice(1, -1) : mem[el]).join('')
      , cmp: (a, b) => cmp = [Number.isInteger(mem[a]) ? mem[a] : a, mem[b] ? mem[b] : b]
      , jmp: lable => cur = lables[lable]
      , je: lable => cmp[0] == cmp[1] ? cur = lables[lable] : null
      , jne: lable => cmp[0] != cmp[1] ? cur = lables[lable] : null
      , jg: lable => cmp[0] > cmp[1] ? cur = lables[lable] : null
      , jge: lable => cmp[0] >= cmp[1] ? cur = lables[lable] : null
      , jl: lable => cmp[0] < cmp[1] ? cur = lables[lable] : null
      , jle: lable => cmp[0] <= cmp[1] ? cur = lables[lable] : null
      , ret: () => cur = cstack.pop()
      , end: () => {finish = 1; cur = prog.length}
    }
  prog = prog.map((el, i) => el.replace(/;.*/, '').replace(/^\s*(\w+):/, (_, label) => {lables[label] = i; return ''}))
  while (cur < prog.length) {
    prog[cur].replace(/^\s*(\w+)\s*(.*)/, (_, op, arg) => ops[op](...(arg.replace(/;.*/, '').match(/('[^']+'|\w+)/g) || []).map(el => el.trim())))
    cur++
  }
  return finish ? msg : -1
}
```

## 2 kyu Evaluate mathematical expression
https://www.codewars.com/kata/52a78825cdfc2cfc87000005/
```js
var calc = function (ex, e = ex.replace(/\s+/g, '')) {
  ops = [
    [/\(([^()]+)\)/g,               (...v) => calc(v[1]) ]
  , [/^--|([-+*/])--/g,             (...v) => v[1] || '' ]
  , [/(-?[\d.]+)([/*])(-?[\d.]+)/g, (...v) => v[2] == '/' ? v[1]/v[3] :  v[1] * v[3] ]
  , [/(-?[\d.]+)([-+])(-?[\d.]+)/g, (...v) => v[2] == '-' ? v[1]-v[3] : +v[1]+ +v[3] ]
  ]
  ops.map(op => { while (e.match(op[0])) e = e.replace(op[0], op[1]) })
  return +e
};
```

## 2 kyu Whitespace Interpreter
https://www.codewars.com/kata/52dc4688eca89d0f820004c6/
```js
function whitespace(code, input) {
  if (!(code = code.replace(/[^\s]+/g, '')).length) throw new Error ('invalid code')
  var marks = {}, calls = [], ptr, output = '', stack = [], heap = {};
  
  var vars = {
    num: () => { if (code[ptr] == "\n") throw new Error ('invalid number'); let num, eon = code.slice(ptr).indexOf("\n");
                 num = parseInt( (code[ptr] == "\t"?'-0':'0') + code.slice(++ptr, eon+ptr).replace(/./g, (s) => s=="\t" ? 1:0), 2);
                 ptr += eon; return num; },
    nth: (num) => { if (num < 0 || stack.length <= num) throw new Error ('out of bound'); return stack[stack.length - 1 - num] },
    a: ()      => { if (!stack.length) throw new Error ('empty stack'); return stack.pop() },
    b: ()      => { if (!stack.length) throw new Error ('empty stack'); return stack.pop() },
    ha: (a)    => { if (!heap.hasOwnProperty(a)) throw new Error ('undefined heap address'); return heap[a] },
    chr: ()    => { if (!input) throw new Error ('end of input'); let chr = input.charCodeAt(0); input = input.slice(1); return chr },
    stn: ()    => { if (!input) throw new Error ('end of input'); let ofs = input.includes("\n") ? input.indexOf("\n") : input.length
                  , num = parseInt(input.slice(0, ofs)); input = input.slice(ofs+1); return num },
    lbn: ()    => { let eol = code.slice(ptr).indexOf("\n"), lbl = code.slice(ptr, eol + ptr); ptr += eol + 1; return lbl },
    lbl: (lbn) => { if (!marks[lbn]) throw new Error ('unknown label'); return lbn }
  },
  commands = [
    // stack
    ["  ",       'pushNum',  (num)    => stack.push( num ) ],
    [" \t ",     'dubVal',   (nth)    => stack.push( nth ) ],
    [" \t\n",    'dropMany', (num)    => stack = stack.slice(0, -1 - (num<0?-1:num)).concat( stack[stack.length-1] ) ],
    [" \n ",     'dubTop',   (a)      => stack = stack.concat(a, a) ],
    [" \n\t",    'swap',     (a, b)   => stack = stack.concat(a, b) ],
    [" \n\n",    'drop',     (a)      => {} ],
    // math
    ["\t   ",    'add',      (a, b)   => stack.push(b + a) ],
    ["\t  \t",   'substr',   (a, b)   => stack.push(b - a) ],
    ["\t  \n",   'multpl',   (a, b)   => stack.push(b * a) ],
    ["\t \t ",   'divide',   (a, b)   => { if (!a) throw new Error ('division by zero');
                                           stack.push(Math.floor(b / a)) }],
    ["\t \t\t",  'rest',     (a, b)   => { if (!a) throw new Error ('division by zero');
                                           stack.push((a < 0 && b > 0 || a > 0 && b < 0 ? a : 0) + b % a) }],
    // heap
    ["\t\t ",    'set',      (a, b)   => heap[b] = a ],
    ["\t\t\t",   'get',      (ha)     => stack.push( ha ) ],
    // io
    ["\t\n  ",   'printChr', (a)      => output += String.fromCharCode( a ) ],
    ["\t\n \t",  'printNum', (a)      => output += a ],
    ["\t\n\t ",  'readChr',  (a, chr) => heap[a] = chr ],
    ["\t\n\t\t", 'readNum',  (a, stn) => heap[a] = stn ],
    // flow
    ["\n  ",     'mark',     (lbn)    => { if (!marks.done && marks[lbn]) throw new Error ('repeated labels');
                                           marks[lbn] = ptr }],
    ["\n \t",    'call',     (lbl)    => { calls.push(ptr); ptr = marks[lbl] }],
    ["\n \n",    'jump',     (lbl)    => ptr = marks[lbl] ],
    ["\n\t ",    'jumpIf0',  (lbl, a) => { if (a == 0) ptr = marks[lbl] }],
    ["\n\t\t",   'jumpIf-',  (lbl, a) => { if (a < 0) ptr = marks[lbl] }],
    ["\n\t\n",   'return',   ()       => { if (!calls.length) throw new Error ('return outside of subroutine');
                                           ptr = calls.pop() }],
    ["\n\n\n",   'exit',     ()       => { marks.exited = true; ptr = code.length }]
  ],
  getArgs = (cmd) => cmd.toString().match(/^\(([^)]*)/)[1].replace(/\s*/g, '')
  
  invoke = (cmd) => {
    let args = getArgs(cmd).split(',').filter((v) => v)
    .map(v => { if (!vars[v]) throw new Error ('invalid variable handle: ' + v); return invoke(vars[v], 1) })
    return cmd(...args)
  }

  [0, 1].forEach( (flag) => {
    ptr = 0
    while (ptr < code.length) {
      let cmd = commands.find((c) => c[0] == code.slice(ptr, ptr + c[0].length)); if (!cmd) throw new Error ('unknown command')
      ptr += cmd[0].length
      if (!flag && getArgs(cmd[2]).match(/lbl|num|nth/)) ptr += code.slice(ptr).indexOf("\n") + 1
      if (flag || cmd[1] == 'mark') invoke(cmd[2])
    }
    marks.done = true
  })

  if (!marks.exited) throw new Error ('unclean termination'); return output;
};
```
