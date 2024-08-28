# Booth_multiplier


module bit16multiplier (
    input clk,
    input [15:0] a,
    input [15:0] b,
    output reg [31:0] product
);

    wire [7:0] product0, product1, product2, product3,product4,product5,product6,product7,product8product9,product10,product11,product12,product13,product14,product15;

    // Instantiate 4-bit multipliers
    bit4multiplier mult0 (.q(a[3:0]),.clk(clk),   .m(b[3:0]),   .c(product0));
    bit4multiplier mult1 (.q(a[3:0]), .clk(clk),  .m(b[7:4]),   .c(product1));
    bit4multiplier mult2 (.q(a[3:0]), .clk(clk), .m(b[11:8]),  .c(product2));
    bit4multiplier mult3 (.q(a[3:0]),.clk(clk), .m(b[15:12]), .c(product3));
    bit4multiplier mult4 (.q(a[7:4]),.clk(clk),   .m(b[3:0]),   .c(product4));
    bit4multiplier mult5 (.q(a[7:4]), .clk(clk),  .m(b[7:4]),   .c(product5));
    bit4multiplier mult6 (.q(a[7:4]),.clk(clk),   .m(b[11:8]),   .c(product6));
    bit4multiplier mult7 (.q(a[7:4]), .clk(clk),  .m(b[15:12]),   .c(product7));
    bit4multiplier mult8 (.q(a[11:8]),.clk(clk),   .m(b[3:0]),   .c(product8));
    bit4multiplier mult9 (.q(a[11:8]), .clk(clk),  .m(b[7:4]),   .c(product9));
    bit4multiplier mult10 (.q(a[11:8]), .clk(clk),  .m(b[11:8]),   .c(product10));
    bit4multiplier mult20 (.q(a[11:8]), .clk(clk),  .m(b[15:12]),   .c(product11));
    bit4multiplier mult30 (.q(a[15:12]),.clk(clk),   .m(b[3:0]),   .c(product12));
    bit4multiplier mult40 (.q(a[15:12]), .clk(clk),  .m(b[7:4]),   .c(product13));
    bit4multiplier mult60 (.q(a[15:12]),.clk(clk),   .m(b[11:8]),   .c(product14));
    bit4multiplier mult70 (.q(a[15:12]),.clk(clk),   .m(b[15:12]),   .c(product15));

    // Pipelining registers
    reg [31:0] stage1_0, stage1_1, stage1_2, stage1_3, stage1_4, stage1_5, stage1_6, stage1_7, stage1_8, stage1_9, stage1_10, stage1_11, stage1_12, stage1_13, stage1_14, stage1_15;
    reg [31:0] stage2_0, stage2_1,stage2_2,stage2_3;
    reg [31:0] stage3_1,stage3_2;
    reg [31:0] stage4;

    always @(posedge clk) begin
            // Stage 1: Capture partial products
            stage1_0 <= {24'b0, product0};
            stage1_1 <= {20'b0, product1, 4'b0};
            stage1_2 <= {16'b0, product2, 8'b0};
            stage1_3 <= {12'b0, product3, 12'b0};
            stage1_4 <= {20'b0, product4, 4'b0};
            stage1_5 <= {16'b0, product5, 8'b0};
            stage1_6 <= {12'b0, product6, 12'b0};
            stage1_7 <= {8'b0, product7, 16'b0};
            stage1_8 <= {16'b0, product8, 8'b0};
            stage1_9 <= {12'b0, product9, 12'b0};
            stage1_10 <= {8'b0, product10, 16'b0};
            stage1_11 <= {4'b0, product11, 20'b0};
            stage1_12 <= {12'b0, product12, 12'b0};
            stage1_13 <= {8'b0, product13, 16'b0};
            stage1_14 <= {4'b0, product14, 20'b0};
            stage1_15 <= {product15, 24'b0};
            stage2_0 <= stage1_0 + stage1_1+stage1_2 + stage1_3;
            stage2_1 <= stage1_4 + stage1_5 + stage1_6 + stage1_7;
            stage2_2 <= stage1_8 + stage1_9 + stage1_10 + stage1_11;
            stage2_3 <= stage1_12 + stage1_13 + stage1_14 + stage1_15;
            stage3_1<= stage2_0+stage2_1;
            stage3_2<= stage2_2+stage2_3;
            stage4 <= stage3_1+stage3_2;
            product<=stage4;
        end
endmodule

module bit4multiplier(
    input [3:0] q,
    input [3:0] m,
    input clk,
    output reg [7:0] c
);
    reg [3:0] l12_q, l23_q;
    reg [3:0] l12_m, l23_m;
    reg [3:0] l12_a, l23_a;
    reg [3:0] l_temp;
    reg [2:0] counter;
    reg qm1, qm_1;

    always @(posedge clk) begin
        l12_q <= q;
        l12_m <= m;
        l12_a <= 4'b0000;
        qm_1 <= 1'b0;
    end

    always @(posedge clk) begin
        l23_q <= l12_q;
        l23_m <= l12_m;
        l23_a <= l12_a;
        qm1 <= qm_1;
        for (counter = 0; counter < 4; counter = counter + 1) begin 
            if ({l23_q[0], qm1} == 2'b01) begin
                l23_a = l23_a + l23_m;
            end else if ({l23_q[0], qm1} == 2'b10) begin
                l23_a = l23_a - l23_m;
            end
            qm1 = l23_q[0];
            l23_q = l23_q >> 1;
            l23_q[3] = l23_a[0];
            l23_a = {l23_a[3], l23_a[3:1]};
        end
    end

    always @(posedge clk) begin
        c <= {l23_a, l23_q};
    end
