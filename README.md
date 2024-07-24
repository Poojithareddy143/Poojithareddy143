module spi_controller(    input clk,
    input rst,
    input [7:0] data_in,
    input mosi_en,
    output reg mosi,
    input miso,
    output [7:0] data_out,
    output spi_done
);

reg [2:0] spi_state;
reg [2:0] bit_cnt;
reg [7:0] data_reg;

parameter IDLE = 3'b000;
parameter TRANSFER = 3'b001;
parameter DONE = 3'b010;

always @(posedge clk or posedge rst) begin
    if (rst) begin
        spi_state <= IDLE;
        bit_cnt <= 0;
        data_reg <= 0;
        mosi <= 0;
        spi_done <= 0;
    end else begin
        case (spi_state)
            IDLE: begin
                if (mosi_en) begin
                    spi_state <= TRANSFER;
                    bit_cnt <= 7;
                    data_reg <= data_in;
                    mosi <= data_reg[7];
                end
            end
            TRANSFER: begin
                if (bit_cnt == 0) begin
                    spi_state <= DONE;
                    spi_done <= 1;
                end else begin
                    bit_cnt <= bit_cnt - 1;
                    data_reg <= {data_reg[6:0], miso};
                    mosi <= data_reg[7];
                end
            end
            DONE: begin
                spi_state <= IDLE;
                spi_done <= 0;
                data_out <= data_reg;
            end
        endcase
    end
end
endmodule      
Testbench:
module tb_spi_controller;
reg clk;
reg rst;
reg [7:0] data_in;
reg mosi_en;
wire mosi;
reg miso;
wire [7:0] data_out;
wire spi_done;
spi_controller sc(.clk(clk), .rst(rst), .data_in(data_in), .mosi_en(mosi_en), .mosi(mosi), .miso(miso), .data_out(data_out), .spi_done(spi_done));initial begin
    clk = 0;
    rst = 1;
    #10 rst = 0;
    #10 data_in = 8'hAA;
    mosi_en = 1;
    #10 mosi_en = 0;
    #100 miso = 1;
    #10 miso = 0;
end
always #5 clk = ~clk;
endmodule
