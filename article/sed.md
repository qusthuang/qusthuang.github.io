###1. sed 行首加字符
sed "s/^/**newstr**/" test1.TXT >>new.txt

###2. sed 行尾加字符
sed "s/$/**newstr**/" sql.txt>>new.txt

###3. sed 替换
sed "s/aaa/**newstr**/g" test.txt>>new.txt

###4. sed 特殊字符
sed "s/aaa/**\\\`bbbb\\\`**/g" test.txt>>new.txt

###5. 首尾各加字符
sed '{s/^/**newstr1**/;s/$/**newstr2**/}' sed.txt >>new.txt
