# Verilog 学习笔记

## 1 Verilog基础

1.  对于wire类型的变量，可以直接用assign 赋值，也可以直接用wire直接声明变量:

   ```verilog
   module top_module( 
       input a,b,c,
       output w,x,y,z );
       assign a = b;
       wire tmp;
       assign tmp = b;
   
   endmodule
   ```



2. 向量（Vector）

   可以声明不同长度的向量:

   ```verilog
   wire [99:0] my_vector;      // Declare a 100-element vector
   assign out = my_vector[10]; // Part-select one bit out of the vector
   ```

   可以用组合数组表示保护向量的向量
   
   ```verilog
   reg [7:0] mem [255:0];   // 256 个元素, 每个元素是个8位的向量
   reg mem2 [28:0];         // 29 个元素,每个元素是个1位的标量
   ```
   
   选取向量时可以使用切片的技术，类似于Python
   
   ```verilog
   w[3:0]
   x[-1:-2] //最低的两位
   ```
   
   可以用{}来连接向量，如用字节序逆转一个31bits的向量:
   
   ````verilog
   module top_module( 
       input [31:0] in,
       output [31:0] out );//
       assign out = {in[7:0], in[15:8], in[23:16], in[31:24]};
      
   endmodule
   ````
   
   可以用位数+‘+进制+值的格式表示一个数字，并可以使用大括号复制
   
   ```verilog
   3'b111 // => 1个3位的向量
   4'd10  // => 1个4位的向量，数值是10
   4'ha   // => 1个4位的向量，数值是a
   // {num{vector}} 注意！这两对{}是必须的！
   {5{1'b1}} // => 5'b11111
   ```



## 2 Verilog 模块

1. 调用一个模块的方法:

   ```verilog
   mod_a instance1 ( wa, wb, wc ); //形参和实参位置必须一样
   mod_a instance2 ( .out(wc), .in1(wa), .in2(wb) ); //位置可以不一样，但以名称对应
   ```

2. case语句智能存在于always块中

   ```verilog
   always @(*) begin
           case (sel)
               2'b00: q = d;
               2'b01: q = a;
               2'b10: q = b;
               2'b11: q = c;
           endcase
   end
   ```

3. 对于不需要传递的参数可以直接省略掉：

   ```verilog
   //如用16位加法器实现32位加法器，高位加法不用保存进位
   add16 low(a[15:0], b[15:0], 1'b0, lowsum, carry);
   add16 hi(a[31:16], b[31:16], carry, highsum,); //这里省略掉了最后一位
   ```

   

## 3 过程块

1. reg 和wire:

   在 always 块外面只能用assign语句给wire类型的变量赋值，在always里面只能对reg类型的变量赋值。

   ```verilog
   wire out1;
   reg out2;
   assign out1 = a & b | c ^ d;
   always @(*) out2 = a & b | c ^ d;
   ```

2. 阻塞赋值与非阻塞赋值:

   在always块中，可以使用阻塞赋值和非阻塞赋值。阻塞是顺序赋值，而非阻塞是并行的，即直到触发条件才开始并行赋值。

   ```verilog
   a = b ; // 阻塞赋值
   a <= b; // 非阻塞赋值
   ```

3. If语句

   ```verilog
   always @(*) begin
       if (condition) begin
           out = x;
       end
       else begin
           out = y;
       end
   end
   //等价于在block外面
   assign out = (condition) ? x : y;
   ```

4. casez语句可以识别带有z的数字，z可以表示任何数字，即系统不关心:

   ```verilog
   always @(*) begin
       casez (in[3:0])
           4'bzzz1: out = 0;   // in[3:1] can be anything
           4'bzz1z: out = 1;
           4'bz1zz: out = 2;
           4'b1zzz: out = 3;
           default: out = 0;
       endcase
   end
   ```



## 4 语言特性

1. 运算符折叠:

   对于一个向量如果计算它每一位的and值可以直接用前缀运算符表示:

   ```verilog
   & a[3:0]     // AND: a[3]&a[2]&a[1]&a[0]. Equivalent to (a[3:0] == 4'hf)
   | b[3:0]     // OR:  b[3]|b[2]|b[1]|b[0]. Equivalent to (b[3:0] != 4'h0)
   ^ c[2:0]     // XOR: c[2]^c[1]^c[0]
   ```

   