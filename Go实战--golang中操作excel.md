## Go实战--golang中操作excel

原创一蓑烟雨1989 最后发布于2017-10-09 13:08:02 阅读数 30722  收藏
展开
生命不止，继续 go go go !!!

匆匆十一假期，继续go，北京阴雨连连：
渲染了一种悲凉的气氛；
暗示了人物双假结束的悲惨命运；
揭露了下半年再无假期的凄惨社会环境；
为假期后各种辛苦的工作埋下伏笔；
与美好的假期形成鲜明的对比。

今天，与大家分享一下golang中如何操作xlsx文件，也就是我们所说的excel。

xlsx简介
A file with the XLSX file extension is a Microsoft Excel Open XML Format Spreadsheet file. It’s an XML-based spreadsheet file created by Microsoft Excel version 2007 and later.

XLSX files organize data in cells that are stored in worksheets, which are in turn stored in workbooks, which are files that contain multiple worksheets. The cells are positioned by rows and columns and can contain styles, formatting, math functions, and more.

Microsoft Office EXCEL 2007/2010/2013/2016文档的扩展名。xlsx是从Office2007开始使用的，是用新的基于XML的压缩文件格式取代了其目前专有的默认文件格式，在传统的文件名扩展名后面添加了字母x（即：docx取代doc、.xlsx取代xls等等），使其占用空间更小。

tealeg/xlsx
github地址：
https://github.com/tealeg/xlsx

Star: 1954

获取：
go get github.com/tealeg/xlsx

有点excel基础的人，都应该清楚什么是sheet，什么是row，cell即内容。

读取xlsx
package main

import (
    "fmt"

    "github.com/tealeg/xlsx"
)

func main() {
    excelFileName := "test.xlsx"
    xlFile, err := xlsx.OpenFile(excelFileName)
    if err != nil {
        fmt.Printf("open failed: %s\n", err)
    }
    for _, sheet := range xlFile.Sheets {
        fmt.Printf("Sheet Name: %s\n", sheet.Name)
        for _, row := range sheet.Rows {
            for _, cell := range row.Cells {
                text := cell.String()
                fmt.Printf("%s\n", text)
            }
        }
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
读取结果：
Sheet Name: Sheet1
姓名
年龄
狗子
18
蛋子
28

创建xlsx
package main

import (
    "fmt"

    "github.com/tealeg/xlsx"
)

func main() {
    var file *xlsx.File
    var sheet *xlsx.Sheet
    var row, row1, row2 *xlsx.Row
    var cell *xlsx.Cell
    var err error

    file = xlsx.NewFile()
    sheet, err = file.AddSheet("Sheet1")
    if err != nil {
        fmt.Printf(err.Error())
    }
    row = sheet.AddRow()
    row.SetHeightCM(1)
    cell = row.AddCell()
    cell.Value = "姓名"
    cell = row.AddCell()
    cell.Value = "年龄"
    
    row1 = sheet.AddRow()
    row1.SetHeightCM(1)
    cell = row1.AddCell()
    cell.Value = "狗子"
    cell = row1.AddCell()
    cell.Value = "18"
    
    row2 = sheet.AddRow()
    row2.SetHeightCM(1)
    cell = row2.AddCell()
    cell.Value = "蛋子"
    cell = row2.AddCell()
    cell.Value = "28"
    
    err = file.Save("test_write.xlsx")
    if err != nil {
        fmt.Printf(err.Error())
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
修改xlsx
package main

import (
    "github.com/tealeg/xlsx"
)

func main() {
    excelFileName := "test.xlsx"
    xlFile, err := xlsx.OpenFile(excelFileName)
    if err != nil {
        panic(err)
    }
    first := xlFile.Sheets[0]
    row := first.AddRow()
    row.SetHeightCM(1)
    cell := row.AddCell()
    cell.Value = "铁锤"
    cell = row.AddCell()
    cell.Value = "99"

    err = xlFile.Save(excelFileName)
    if err != nil {
        panic(err)
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
Luxurioust/excelize或360EntSecGroup-Skylar/excelize
github地址：
https://github.com/360EntSecGroup-Skylar/excelize

Star: 1476

获取：
go get github.com/xuri/excelize

读取xlsx
package main

import (
    "fmt"

    "github.com/xuri/excelize"
)

func main() {
    xlsx, err := excelize.OpenFile("test.xlsx")
    if err != nil {
        fmt.Println(err)
        return
    }
    cell := xlsx.GetCellValue("Sheet1", "B2")
    fmt.Println(cell)

    rows := xlsx.GetRows("Sheet1")
    for _, row := range rows {
        for _, colCell := range row {
            fmt.Print(colCell, "\t")
        }
        fmt.Println()
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
输出：
18
姓名 年龄
狗子 18
蛋子 28
大便 99

创建xlsx
package main

import (
    "fmt"

    "github.com/xuri/excelize"
)

func main() {
    xlsx := excelize.NewFile()

    index := xlsx.NewSheet("Sheet1")
    xlsx.SetCellValue("Sheet1", "A1", "姓名")
    xlsx.SetCellValue("Sheet1", "B1", "年龄")
    xlsx.SetCellValue("Sheet1", "A2", "狗子")
    xlsx.SetCellValue("Sheet1", "B2", "18")
    // Set active sheet of the workbook.
    xlsx.SetActiveSheet(index)
    // Save xlsx file by the given path.
    err := xlsx.SaveAs("test_write.xlsx")
    if err != nil {
        fmt.Println(err)
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
插入图表
package main

import (
    "fmt"

    "github.com/xuri/excelize"
)

func main() {
    categories := map[string]string{"A2": "Small", "A3": "Normal", "A4": "Large", "B1": "Apple", "C1": "Orange", "D1": "Pear"}
    values := map[string]int{"B2": 2, "C2": 3, "D2": 3, "B3": 5, "C3": 2, "D3": 4, "B4": 6, "C4": 7, "D4": 8}
    xlsx := excelize.NewFile()
    for k, v := range categories {
        xlsx.SetCellValue("Sheet1", k, v)
    }
    for k, v := range values {
        xlsx.SetCellValue("Sheet1", k, v)
    }
    xlsx.AddChart("Sheet1", "E1", `{"type":"bar3D","series":[{"name":"=Sheet1!$A$2","categories":"=Sheet1!$B$1:$D$1","values":"=Sheet1!$B$2:$D$2"},{"name":"=Sheet1!$A$3","categories":"=Sheet1!$B$1:$D$1","values":"=Sheet1!$B$3:$D$3"},{"name":"=Sheet1!$A$4","categories":"=Sheet1!$B$1:$D$1","values":"=Sheet1!$B$4:$D$4"}],"title":{"name":"Fruit 3D Line Chart"}}`)
    // Save xlsx file by the given path.
    err := xlsx.SaveAs("test_write.xlsx")
    if err != nil {
        fmt.Println(err)
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
插入图片
package main

import (
    "fmt"
    _ "image/gif"
    _ "image/jpeg"
    _ "image/png"

    "github.com/xuri/excelize"
)

func main() {
    xlsx, err := excelize.OpenFile("test.xlsx")
    if err != nil {
        fmt.Println(err)
        return
    }
    // Insert a picture.
    err = xlsx.AddPicture("Sheet1", "A2", "image1.png", "")
    if err != nil {
        fmt.Println(err)
    }
    // Insert a picture to worksheet with scaling.
    err = xlsx.AddPicture("Sheet1", "D2", "image2.jpg", `{"x_scale": 0.5, "y_scale": 0.5}`)
    if err != nil {
        fmt.Println(err)
    }
    // Insert a picture offset in the cell with printing support.
    err = xlsx.AddPicture("Sheet1", "H2", "image3.gif", `{"x_offset": 15, "y_offset": 10, "print_obj": true, "lock_aspect_ratio": false, "locked": false}`)
    if err != nil {
        fmt.Println(err)
    }
    // Save the xlsx file with the origin path.
    err = xlsx.Save()
    if err != nil {
        fmt.Println(err)
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39

————————————————
版权声明：本文为CSDN博主「一蓑烟雨1989」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/wangshubo1989/article/details/78181493