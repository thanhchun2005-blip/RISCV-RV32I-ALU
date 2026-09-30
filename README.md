# 32-bit Arithmetic Logic Unit (ALU) - RISC-V RV32I

[![Language](https://img.shields.io/badge/Language-SystemVerilog-blue.svg)](https://en.wikipedia.org/wiki/SystemVerilog)
[![Standard](https://img.shields.io/badge/Standard-IEEE%201800--2012%2F2017-brightgreen.svg)]()
[![Target ISA](https://img.shields.io/badge/ISA-RISC--V%20RV32I-red.svg)](https://riscv.org/)
[![Tool](https://img.shields.io/badge/Verified%20with-Vivado%202022.2-orange.svg)]()
[![License](https://img.shields.io/badge/License-MIT-green.svg)]()

Khối **ALU (Arithmetic Logic Unit)** 32-bit là thành phần tính toán trung tâm trong tầng thực thi (**Execute - EX stage**) của vi xử lý RISC-V RV32I. Module thực hiện toàn bộ 10 phép toán số học, logic và dịch bit cơ sở của tập lệnh RV32I cho cả hai định dạng lệnh **R-type** và **I-type**.

---

## 📌 Đặc tả Chức năng & Sơ đồ Khối

Module nhận hai toán hạng 32-bit ($A$ và $B$) cùng mã điều khiển `ALUControl` 4-bit, xuất ra kết quả `Result` 32-bit một cách tổ hợp (combinational logic):

```
                     +-----------------------+
      A [31:0] ----->|                       |
      B [31:0] ----->|       ALU_core        |-----> Result [31:0]
ALUControl [3:0] --->| (32-bit Combinational)|
                     +-----------------------+
```

### 📋 Bảng Chân trị Mã Phép toán (`ALUControl`)

| `ALUControl[3:0]` | Phép toán tương ứng | Lệnh RISC-V hỗ trợ | Mô tả logic SystemVerilog |
| :---: | :--- | :--- | :--- |
| `4'b0000` | **ADD** | `ADD`, `ADDI` | `Result = A + B` |
| `4'b0001` | **SUB** | `SUB` | `Result = A - B` |
| `4'b0010` | **AND** | `AND`, `ANDI` | `Result = A & B` |
| `4'b0011` | **OR** | `OR`, `ORI` | `Result = A \| B` |
| `4'b0100` | **XOR** | `XOR`, `XORI` | `Result = A ^ B` |
| `4'b0101` | **SLL** | `SLL`, `SLLI` | `Result = A << B[4:0]` (Dịch trái logic) |
| `4'b0110` | **SRL** | `SRL`, `SRLI` | `Result = A >> B[4:0]` (Dịch phải logic) |
| `4'b0111` | **SRA** | `SRA`, `SRAI` | `Result = $signed(A) >>> B[4:0]` (Dịch phải số học) |
| `4'b1000` | **SLT** | `SLT`, `SLTI` | `Result = ($signed(A) < $signed(B)) ? 1 : 0` (So sánh có dấu) |
| `4'b1001` | **SLTU** | `SLTU`, `SLTIU` | `Result = (A < B) ? 1 : 0` (So sánh không dấu) |
| *Mặc định* | *Default* | - | `Result = 32'b0` |

---

## 🔌 Đặc tả Cổng Giao tiếp (Port Specification)

| Tên cổng | Hướng (Direction) | Độ rộng bit | Ý nghĩa |
| :--- | :---: | :---: | :--- |
| `A` | Input | `[31:0]` | Toán hạng 1 (từ thanh ghi `rs1` hoặc `PC`) |
| `B` | Input | `[31:0]` | Toán hạng 2 (từ thanh ghi `rs2` hoặc giá trị tức thời `imm`) |
| `ALUControl` | Input | `[3:0]` | Tín hiệu điều khiển chọn phép toán từ ALU Decoder |
| `Result` | Output | `[31:0]` | Kết quả tính toán của ALU đưa về bộ nhớ hoặc Write-Back |

---

## 🧪 Kiểm chứng & Mô phỏng (Verification)

Module đi kèm testbench tự kiểm tra toàn diện `testbench/tb_alu.sv` kiểm tra đầy đủ:
- Các phép toán cơ bản ADD, SUB, AND, OR, XOR.
- Dịch bit với các lượng dịch `shamt` từ `0` đến `31`.
- Phân biệt dịch số học (SRA) bảo toàn bit dấu và dịch logic (SRL) chèn bit 0.
- Các trường hợp so sánh biên signed/unsigned (`0x80000000`, `0x7FFFFFFF`, `0xFFFFFFFF`, `0x00000000`).

### Lệnh chạy mô phỏng với Vivado Simulator (XSim):

```bash
# Biên dịch file RTL và Testbench
xvlog -sv rtl/alu.sv testbench/tb_alu.sv

# Elaborate
xelab ALU_tb -s alu_sim

# Chạy mô phỏng không cần GUI
xsim alu_sim -R
```

---

## 📂 Cấu trúc Thư mục Repo

```
.
├── rtl/
│   └── alu.sv             # Mã nguồn phần cứng SystemVerilog (ALU_core)
├── testbench/
│   └── tb_alu.sv          # Testbench tự kiểm tra (ALU_tb)
├── .gitignore
└── README.md
```

---

## 👨‍💻 Thông tin Tác giả & Đồ án

- **Sinh viên thực hiện:** Nguyễn Thành Trung
- **Học phần:** Đồ án Môn học 2 (Capstone Project II) – Ngành Kỹ thuật Máy tính
- **Tên đề tài:** Thiết kế, kiểm chứng và triển khai FPGA lõi vi xử lý RISC-V RV32I 32-bit pipeline 5 tầng ở mức RTL bằng SystemVerilog
- **GitHub cá nhân:** [@thanhchun2005-blip](https://github.com/thanhchun2005-blip)
- **Email:** thanhchun2005@gmail.com
