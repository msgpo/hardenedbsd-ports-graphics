--- ./gcc/config/mips/mips.md.orig	2011-03-03 21:56:58.000000000 +0000
+++ ./gcc/config/mips/mips.md	2012-01-21 14:11:19.000000000 +0000
@@ -37,6 +37,7 @@
   74kf2_1
   74kf1_1
   74kf3_2
+  allegrex
   loongson_2e
   loongson_2f
   loongson_3a
@@ -598,7 +599,7 @@
 ;; This mode iterator allows :MOVECC to be used anywhere that a
 ;; conditional-move-type condition is needed.
 (define_mode_iterator MOVECC [SI (DI "TARGET_64BIT")
-                              (CC "TARGET_HARD_FLOAT && !TARGET_LOONGSON_2EF")])
+                              (CC "TARGET_HARD_FLOAT && !TARGET_LOONGSON_2EF && !TARGET_ALLEGREX")])
 
 ;; 32-bit integer moves for which we provide move patterns.
 (define_mode_iterator IMOVE32
@@ -1885,11 +1886,11 @@
 	   (mult:DI
 	      (any_extend:DI (match_operand:SI 1 "register_operand" "d"))
 	      (any_extend:DI (match_operand:SI 2 "register_operand" "d")))))]
-  "!TARGET_64BIT && (ISA_HAS_MSAC || GENERATE_MADD_MSUB || ISA_HAS_DSP)"
+  "!TARGET_64BIT && (ISA_HAS_MSAC || GENERATE_MADD_MSUB || ISA_HAS_DSP || TARGET_ALLEGREX)"
 {
   if (ISA_HAS_DSP_MULT)
     return "msub<u>\t%q0,%1,%2";
-  else if (TARGET_MIPS5500 || GENERATE_MADD_MSUB)
+  else if (TARGET_MIPS5500 || GENERATE_MADD_MSUB || TARGET_ALLEGREX)
     return "msub<u>\t%1,%2";
   else
     return "msac<u>\t$0,%1,%2";
@@ -2066,14 +2067,14 @@
 	 (mult:DI (any_extend:DI (match_operand:SI 1 "register_operand" "d"))
 		  (any_extend:DI (match_operand:SI 2 "register_operand" "d")))
 	 (match_operand:DI 3 "register_operand" "0")))]
-  "(TARGET_MAD || ISA_HAS_MACC || GENERATE_MADD_MSUB || ISA_HAS_DSP)
+  "(TARGET_MAD || ISA_HAS_MACC || GENERATE_MADD_MSUB || ISA_HAS_DSP || TARGET_ALLEGREX)
    && !TARGET_64BIT"
 {
   if (TARGET_MAD)
     return "mad<u>\t%1,%2";
   else if (ISA_HAS_DSP_MULT)
     return "madd<u>\t%q0,%1,%2";
-  else if (GENERATE_MADD_MSUB || TARGET_MIPS5500)
+  else if (GENERATE_MADD_MSUB || TARGET_MIPS5500 || TARGET_ALLEGREX)
     return "madd<u>\t%1,%2";
   else
     /* See comment in *macc.  */
@@ -2500,6 +2501,33 @@
 ;;
 ;;  ....................
 ;;
+;; FIND FIRST BIT INSTRUCTION
+;;
+;;  ....................
+;;
+
+(define_expand "ffs<mode>2"
+  [(set (match_operand:GPR 0 "register_operand" "")
+   (ffs:GPR (match_operand:GPR 1 "register_operand" "")))]
+  "ISA_HAS_CLZ_CLO"
+{
+  rtx r1, r2, r3, r4;
+  
+  r1 = gen_reg_rtx (<MODE>mode);
+  r2 = gen_reg_rtx (<MODE>mode);
+  r3 = gen_reg_rtx (<MODE>mode);
+  r4 = gen_reg_rtx (<MODE>mode);
+  emit_insn (gen_neg<mode>2 (r1, operands[1]));
+  emit_insn (gen_and<mode>3 (r2, operands[1], r1));
+  emit_insn (gen_clz<mode>2 (r3, r2));
+  emit_move_insn (r4, GEN_INT (GET_MODE_BITSIZE (<MODE>mode)));
+  emit_insn (gen_sub<mode>3 (operands[0], r4, r3));
+  DONE;
+})
+
+;;
+;;  ....................
+;;
 ;;	NEGATION and ONE'S COMPLEMENT
 ;;
 ;;  ....................
@@ -2550,6 +2578,25 @@
   [(set_attr "alu_type" "not")
    (set_attr "mode" "<MODE>")])
 
+(define_expand "rotl<mode>3"
+  [(set (match_operand:GPR 0 "register_operand")
+       (rotate:GPR (match_operand:GPR 1 "register_operand")
+           (match_operand:SI 2 "arith_operand")))]
+  "ISA_HAS_ROR"
+{
+  rtx temp;
+
+  if (GET_CODE (operands[2]) == CONST_INT)
+    temp = GEN_INT (GET_MODE_BITSIZE (<MODE>mode) - INTVAL (operands[2]));
+  else
+    {
+      temp = gen_reg_rtx (<MODE>mode);
+      emit_insn (gen_neg<mode>2 (temp, operands[2]));
+    }
+  emit_insn (gen_rotr<mode>3 (operands[0], operands[1], temp));
+  DONE;
+})
+
 ;;
 ;;  ....................
 ;;
@@ -6301,7 +6348,7 @@
 		 (const_int 0)])
 	 (match_operand:GPR 2 "reg_or_0_operand" "dJ,0")
 	 (match_operand:GPR 3 "reg_or_0_operand" "0,dJ")))]
-  "ISA_HAS_CONDMOVE"
+  "ISA_HAS_CONDMOVE || ISA_HAS_INT_CONDMOVE"
   "@
     mov%T4\t%0,%z2,%1
     mov%t4\t%0,%z3,%1"
@@ -6331,8 +6378,12 @@
 	(if_then_else:GPR (match_dup 5)
 			  (match_operand:GPR 2 "reg_or_0_operand")
 			  (match_operand:GPR 3 "reg_or_0_operand")))]
-  "ISA_HAS_CONDMOVE"
+  "ISA_HAS_CONDMOVE || ISA_HAS_INT_CONDMOVE"
 {
+  if (ISA_HAS_INT_CONDMOVE
+      && GET_MODE_CLASS (GET_MODE (operands[0])) == MODE_FLOAT)
+    FAIL;
+
   mips_expand_conditional_move (operands);
   DONE;
 })
@@ -6481,6 +6532,9 @@
 ; ST-Microelectronics Loongson-2E/2F-specific patterns.
 (include "loongson.md")
 
+; Sony ALLEGREX instructions.
+(include "allegrex.md")
+
 (define_c_enum "unspec" [
   UNSPEC_ADDRESS_FIRST
 ])
