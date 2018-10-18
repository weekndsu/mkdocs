# 利用python批量处理word+excel数据

```
import docx
from docx import Document
import xlrd
from docx.shared import Pt
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.shared import Inches

# 定义读取excel的函数
def read_excel(num):
    ExcelFile=xlrd.open_workbook(r'C:\Users\suqiong.77PAY\Desktop\test.xlsx')
    # 打印出所有的sheet names
    print (ExcelFile.sheet_names())
    # 处理索引为num的sheet
    sheet=ExcelFile.sheet_by_index(num)
    # 处理的sheet的name
    wordname=sheet.name
    # 处理的sheet有多少行
    nrows=sheet.nrows
    # 处理的sheet有多少列
    ncols=sheet.ncols
    # 定义一个空的列表
    result=[]
    # 循环取出每一列的值，放入result
    for i in range(sheet.ncols):
        coli=sheet.col_values(i)
        result.append(coli)
    # 函数返回列表，sheet的名称，sheet的行数，sheet的列数
    # 这里sheet的行和列数主要用于后续循环写入word文档的条件
    return result,wordname,nrows,ncols

def write_word(result,wordname,nrows,ncols):
	# 从result里取出值
    num_id = result[0]
    username = result[1]
    cardno = result[2]
    mername = result[3]
    phoneno = result[4]
    categy = result[5]
    lease = result[6]
    ncols = ncols
    nrows = nrows
    # 设置word的title标题
    title= wordname+'证明'
    # 初始化一个doc文件
    doc = docx.Document()
    # 添加一行空行
    doc.add_paragraph('\n')
    t1 = doc.add_paragraph()
    t1_format = t1.paragraph_format
    t1_format.alignment = WD_ALIGN_PARAGRAPH.CENTER
    run = t1.add_run(title)
    font = run.font
    font.bold = True
    # 设置字体样式
    font.name = '宋体'
    # 设置字体大小
    font.size = Pt(16)
    doc.add_paragraph('\n')
    # 添加一个段落
    p1 = doc.add_paragraph(r'位于____________的_________ 农贸市场（以下简称“本市场”）系合法经营、有序管理的农贸市场，我方作为本市场管理方，严格遵守相关规定，不定期巡查、核实并规范管理本市场内设摊商家的具体经营活动。')
    # 设定格式
    paragraph_format = p1.paragraph_format
    p1.space_after = Pt(5)
    p1.space_before = Pt(10)
    # 靠左对齐
    paragraph_format.alignment = WD_ALIGN_PARAGRAPH.LEFT
    # 首行缩进
    paragraph_format.first_line_indent = Inches(0.5)
    p2 = doc.add_paragraph(r'我方确认在本市场内设摊经营的商家均从事合法合规的农产品销售经营活动，包括但不限于蔬菜、肉类、水产品、酒类等农产品的售卖，具体设摊明细详见附件。')
    paragraph_format = p2.paragraph_format
    # 这是行间距
    p2.line_spacing = Pt(20)
    paragraph_format.alignment = WD_ALIGN_PARAGRAPH.LEFT
    paragraph_format.first_line_indent = Inches(0.5)
    p3 = doc.add_paragraph(r'特此证明!')
    paragraph_format = p3.paragraph_format
    paragraph_format.alignment = WD_ALIGN_PARAGRAPH.LEFT
    paragraph_format.left_indent = Inches(0.5)
    doc.add_paragraph('\n')
    p4=doc.add_paragraph(r' 管理方：')
    paragraph_format = p4.paragraph_format
    paragraph_format.alignment = WD_ALIGN_PARAGRAPH.RIGHT
    paragraph_format.right_indent = Inches(0.5)
    doc.add_paragraph('\n')
    p5=doc.add_paragraph(r'年  月')
    paragraph_format = p5.paragraph_format
    paragraph_format.alignment = WD_ALIGN_PARAGRAPH.RIGHT
    paragraph_format.right_indent = Inches(0.5)
    doc.add_paragraph('\n')
    doc.add_paragraph('\n')
    doc.add_paragraph('\n')
    doc.add_paragraph('\n')
    doc.add_paragraph('\n')
    p6=doc.add_paragraph(r' 附件：设摊明细')
    paragraph_format = p6.paragraph_format
    paragraph_format.alignment = WD_ALIGN_PARAGRAPH.LEFT
    paragraph_format.left_indent = Inches(0.5)
    doc.add_paragraph('\n')
    # 初始化table
    table = doc.add_table(rows=nrows, cols=ncols,style='Table Grid')
    # 循环添加数据
    for i in range(nrows):
        hdr_cells = table.rows[i].cells
        hdr_cells[0].text= str(num_id[i])
        hdr_cells[1].text = str(username[i])
        hdr_cells[2].text = str(cardno[i])
        hdr_cells[3].text = str(mername[i])
        hdr_cells[4].text = str(phoneno[i])
        hdr_cells[5].text = str(categy[i])
        hdr_cells[6].text = str(lease[i])
    # 保存文件
    doc.save(r'C:\Users\suqiong.77PAY\Desktop\信息文档\\'+ wordname +'.docx')

if __name__ =='__main__':
    # 设置循环次（处理sheet的个数）
    for num in range(22):
        result,wordname,nrows,ncols = read_excel(num)
        write_word(result,wordname,nrows,ncols)
```

