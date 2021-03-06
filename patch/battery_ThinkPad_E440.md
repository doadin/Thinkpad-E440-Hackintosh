# battery_ThinkPad_E440.txt

# version(版本): 1.0
# Update Time(更新时间) :2017-01-26

# initial work by Zz.Mark 2015-09-21, 由 Zz.Mark 制作 

# works for: ThinkPad E440 20C5A08ECD

# Tip: If you get a 0% battery status,you should also patch the Rehabman's "Fix Mutex with non-zero SyncLevel" patch.
# You also can patch the both of your computer's battery patch and the Rehabman's patch at one time.

# 注意：如果打过电量补丁后，有获取的电池状态显示为0%的情况，你需要打 Rehabman的 “Fix Mutex with non-zero SyncLevel” 补丁。
# 你也可以，一次性打好 自己电脑的电量补丁 和 Rehabman 的那个补丁。

# ==== Method Load ====

into method label B1B2 remove_entry;
into definitionblock code_regex . insert
begin
Method (B1B2, 2, NotSerialized)\n
{\n
Return(Or(Arg0, ShiftLeft(Arg1, 8)))\n
}\n
end;

into method label B1B4 remove_entry;
into definitionblock code_regex . insert
begin
Method (B1B4, 4, NotSerialized)\n
{\n
    Store(Arg3, Local0)\n
    Or(Arg2, ShiftLeft(Local0, 8), Local0)\n
    Or(Arg1, ShiftLeft(Local0, 8), Local0)\n
    Or(Arg0, ShiftLeft(Local0, 8), Local0)\n
    Return(Local0)\n
}\n
end;

into method label RE1B parent_label EC remove_entry;
into method label RECB parent_label EC remove_entry;
into device label EC insert
begin
Method (RE1B, 1, NotSerialized)\n
{\n
    OperationRegion(ERAM, EmbeddedControl, Arg0, 1)\n
    Field(ERAM, ByteAcc, NoLock, Preserve) { BYTE, 8 }\n
    Return(BYTE)\n
}\n
Method (RECB, 2, Serialized)\n
// Arg0 - offset in bytes from zero-based EC\n
// Arg1 - size of buffer in bits\n
{\n
    ShiftRight(Arg1, 3, Arg1)\n
    Name(TEMP, Buffer(Arg1) { })\n
    Add(Arg0, Arg1, Arg1)\n
    Store(0, Local0)\n
    While (LLess(Arg0, Arg1))\n
    {\n
        Store(RE1B(Arg0), Index(TEMP, Local0))\n
        Increment(Arg0)\n
        Increment(Local0)\n
    }\n
    Return(TEMP)\n
}\n
end;

# ==== Field devide ====

into device label EC code_regex SBRC,\s+16, replace_matched begin SBR1,8,SBR2,8, end;
into device label EC code_regex SBAC,\s+16, replace_matched begin SBC1,8,SBC2,8, end;
into device label EC code_regex SBVO,\s+16, replace_matched begin SVO1,8,SVO2,8, end;
into device label EC code_regex SBBM,\s+16, replace_matched begin SBM1,8,SBM2,8, end;
into device label EC code_regex SBDC,\s+16, replace_matched begin SDC1,8,SDC2,8, end;
into device label EC code_regex SBDV,\s+16, replace_matched begin SDD1,8,SDD2,8, end;
into device label EC code_regex SBSN,\s+16 replace_matched begin SBN1,8,SBN2,8, end;
into device label EC code_regex SBFC,\s+16, replace_matched begin SBF1,8,SBF2,8, end;
into device label EC code_regex SBCH,\s+32 replace_matched begin SCH0,8,SCH1,8,SCH2,8,SCH3,8 end;

# ==== Replace ====

into method label GBST  code_regex SBRC replace_matched begin B1B2(SBR1,SBR2) end;
into method label GBST  code_regex SBRC replace_matched begin B1B2(SBR1,SBR2) end;
into method label GBST  code_regex SBAC replace_matched begin B1B2(SBC1,SBC2) end;
into method label GBST  code_regex SBVO replace_matched begin B1B2(SVO1,SVO2) end;
into method label GBIF  code_regex SBBM replace_matched begin B1B2(SBM1,SBM2) end;
into method label GBIF  code_regex SBDC replace_matched begin B1B2(SDC1,SDC2) end;
into method label GBIF  code_regex SBDC replace_matched begin B1B2(SDC1,SDC2) end;
into method label GBIF  code_regex SBDV replace_matched begin B1B2(SDD1,SDD2) end;
into method label GBIF  code_regex SBDV replace_matched begin B1B2(SDD1,SDD2) end;
into method label GBIF  code_regex SBDV replace_matched begin B1B2(SDD1,SDD2) end;
into method label GBIF  code_regex SBSN replace_matched begin B1B2(SBN1,SBN2) end;
into method label GBIF  code_regex SBFC replace_matched begin B1B2(SBF1,SBF2) end;
into method label GBIF  code_regex SBFC replace_matched begin B1B2(SBF1,SBF2) end;

into method label GBIF code_regex SBCH replaceall_matched begin B1B4(SCH0,SCH1,SCH2,SCH3) end;

# utility methods to read/write buffers from/to EC

into method label GBIF code_regex SBMN replaceall_matched begin RECB(0xA0, 128) end;
into method label GBIF code_regex SBDN replaceall_matched begin RECB(0xA0, 128) end;
