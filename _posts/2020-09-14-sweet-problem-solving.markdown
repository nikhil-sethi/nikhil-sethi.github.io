---
layout: post
title: "Sweet Problem Solving"
date: 2020-09-14
categories: problem-solving vba moms-are-the-best macros
---

<p>Hello again.&nbsp;</p>
<p>I've been very eager to start writing some technical stuff on this blog. I worked on a very small VBA project
    recently. Let's start with some introduction.</p>
<p>I love problem-solving. I'd like to think I'm good at it. But its probably not the technical aptitude that makes me
    good at it (It's a significant part though). My agility and unwavering desire to make everything better is the
    primary factor.</p>
<p>My mother is a kindergarten teacher in a school that requires her to write lengthy report cards for 30-40 children
    twice a year in a very short span of 1-2 weeks. She wanted to be an interior designer. THIS is a problem. Much more
    deep-seated than it seems.&nbsp;<br />When I was in school, when my bent of mind was more towards
    building/fabricating stuff, I used to mentally solve this problem by designing a CNC machine with a G-code to write
    the text. I had even thought of the caveats like repetitive handwriting styles etc. But of course, it was never
    realized.</p>
<p>This year presented itself with an opportunity. The report cards were being made online and the teachers had to type
    them on Microsoft word. My mother is not very computer savvy and not used to typing a lot. THIS was a problem. A
    little intro to the problem:</p>
<p></p>
<ol style="text-align: left;">
    <li>The report card has a series of questions and a text box below to answer them.</li>
    <li>There is already a prefilled comments bank containing three variants of answers depending on the rating of the
        student- Excellent, Good, and Average.</li>
    <li>There are other details like each students name, admin number, photo, etc. that need to be filled manually<br />
    </li>
</ol><i>Disclaimer: I won't be able to provide any snapshots of the final work and word doc of course to avoid any legal
    clash with the school. The full code is given though.</i><br /><br />
<div>Onto the solution. I had recently worked on using macros to automate a CAD design on FreeCAD with Python. And I
    knew that word had support for macros. However, I didn't know anything about visual basic before this of course. So
    I began testing.<p></p>
    <p>The first thing that came to mind as a solution was a macro button along with each box that could fill the box
        according to the child from the comments bank. This would reduce the number of copy-pastes and mouse movements.
        But it was still too manual. Regardless of the current quality of the solution, I knew that a macro to fill text
        boxes from a source was needed for any solution. After some research, the following visual basic code from SoF
        with some modifications worked fine for setting any property of any text box.</p>
    <div style="background: rgb(240, 240, 240); overflow: auto; padding: 0.5em 0.6em; width: auto;">
        <pre
            style="line-height: 125%; margin: 0px;">Set oShape = ActiveDocument.Shapes(j)&nbsp;&nbsp; &nbsp;'<i>j is the number of the shape</i></pre>
        <pre style="line-height: 125%; margin: 0px;">If oShape.AutoShapeType = msoShapeRectangle Then '<i>we need to check both if oShape is of type msoShapeRectangle and its textframe contains place for writing</i>
    If oShape.TextFrame.HasText = True Then
        oShape.TextFrame.TextRange.Text = "Insert your text here"
        ' other properties can also be changed
        oShape.TextFrame.TextRange.Font.Name = "Arial"
        oShape.TextFrame.TextRange.ParagraphFormat.LineSpacing = LinesToPoints(1.5)
        oShape.TextFrame.TextRange.Font.Size = 12
    End If
End If
</pre>
    </div>
    <p>Now I needed a consolidated place to get my text from and place it in the line. Using word as itself as an
        interface was not viable as the comment answers cannot be stored into separate places and I would have to define
        a method to get the end and start of each line. Not to mention its place in the doc as well. Plus I had better
        future plans for all this, so there had to be a common database for everything. I needed something more concrete
        and Excel was just the place for it.&nbsp; So, onto interface testing.</p>
    <p>There are two ways to do this of course. Either run the macro from excel with word as an application object or
        the other ways round. I chose the latter because there was a preexisting report card template where I was
        already doing my testing. To get Excel as an application object and get the text from an existing worksheet:</p>
    <div style="background: rgb(240, 240, 240); overflow: auto; padding: 0.5em 0.6em; width: auto;">
        <pre style="line-height: 125%; margin: 0px;">Dim objExcel As Object 
Set objExcel = CreateObject("Excel.Application")
Set wb = Workbooks.Open("<i>path_to_your_excel_file</i>")</pre>
        <pre style="line-height: 125%; margin: 0px;">Set c_list = wb.Sheets("<i>name_of_your_sheeet</i>")
</pre>
        <pre style="line-height: 125%; margin: 0px;"><br /></pre>
        <pre
            style="line-height: 125%; margin: 0px;">MsgBox c_list.Range("A1").Value&nbsp;&nbsp; &nbsp;'<i>to check if its working</i></pre>
    </div>
    <p>The next step was linking the above two snippets together. At this point, I was pretty clear on what I wanted the
        end product to look like. I decided to go for full automation by achieving the following targets on the click of
        a button:</p>
    <p></p>
    <ol style="text-align: left;">
        <li>All report cards for all students to be created from the main template</li>
        <li>The necessary details like roll number etc. to be sourced from a source excel sheet</li>
        <li>The algorithm should take each child's rating and select from that "rating's" answer bank randomly and fill
            the appropriate text box accordingly.&nbsp;</li>
    </ol>The pseudocode for the above looks like:<p></p>
    <p><i>for each student upto number of students<br /><span>&nbsp;&nbsp; &nbsp;</span>' create document copy, set
            docname = student name<br /><span>&nbsp;&nbsp; </span>&nbsp;' make copy active<br /><span>&nbsp;&nbsp;
                &nbsp;</span>' get child rating from sheet<br /><span>&nbsp;&nbsp; &nbsp;</span>' set initial and final
            box default values like admission number etc.<br /><span>&nbsp;&nbsp; &nbsp;</span>' for each
            textbox<br /><span>&nbsp;&nbsp; </span>&nbsp;<span>&nbsp;&nbsp; &nbsp;</span>' set text of jth text box to
            random column from jth row of 'student_rating' sheet.<br /><span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;
                &nbsp; ' set necessary properties like font and line spacing of text box</span><br /></i></p>
    <p><i>save active document<br />close active document</i></p>
    <p>The first statement under the first loop creates, names, and opens a copy of the current template document. This
        also makes the Word Doc active to do operations on. The following code achieves this.</p>
    <div style="background: rgb(240, 240, 240); overflow: auto; padding: 0.5em 0.6em; width: auto;">
        <pre style="line-height: 125%; margin: 0px;"><pre style="line-height: 18.7501px; margin-bottom: 0px; margin-top: 0px;">student_name = c_list.Range("C" &amp; i + 1).Value&nbsp;&nbsp; 
student_rating = c_list.Range("E" &amp; i + 1).Value
    
save_source = "<i>document_save_location</i>"
doc_name = student_name &amp; ".docx"
    
'Copy doc
WordBasic.CopyFileA FileName:=save_source &amp; ActiveDocument.Name, _
Directory:=save_source &amp; doc_name
'Open and Make active
Documents.Open (save_source &amp; doc_name)</pre>
        </pre>
    </div>
    <p>The next step is the randomizing algorithm. The objective is to take the student's name, find his/her rating from
        a "class_list" sheet, take the current question/text number and then select a random column's value from that
        row. The one line below when put under the second loop does exactly that.</p>
    <div style="background: rgb(240, 240, 240); overflow: auto; padding: 0.5em 0.6em; width: auto;">
        <pre style="line-height: 125%; margin: 0px;">'j:1--&gt;number of text boxes</pre>
        <pre
            style="line-height: 125%; margin: 0px;"><span>&nbsp;&nbsp; &nbsp;</span>oShape.TextFrame.TextRange.Text = wb.Sheets(student_rating).Cells(j - 15, Int(1 + Rnd * (5 - 1 + 1))).Value</pre>
    </div>
    <p>-j is the iterator on the text boxes.<br />-I used Cells(x,y) instead of Range("") because I didn't want to
        convert normal numbers to ASCII characters<br />-The 5 is the maximum number of comment variations that an
        answer to a single question can have. The more the variations, the more different will each report card will
        be.<br />-The 15 is just the number of the textbox from where the questions start. Oh, and I couldn't find any
        method in the GUI which told me the shape number. So what I did was simply print the index 'j' as text inside
        the shape after setting the current shape to that same index (like in the first code in the blog)</p>
    <div style="background: rgb(240, 240, 240); overflow: auto; padding: 0.5em 0.6em; width: auto;">
        <pre
            style="line-height: 125%; margin: 0px;"><pre style="line-height: 18.7501px; margin-bottom: 0px; margin-top: 0px;">'inside second loop</pre>
        <pre
            style="line-height: 18.7501px; margin-bottom: 0px; margin-top: 0px;">Set oShape = ActiveDocument.Shapes(j)</pre>
        <pre
            style="line-height: 18.7501px; margin-bottom: 0px; margin-top: 0px;">'if conditions to check for text writing</pre>
        <pre
            style="line-height: 18.7501px; margin-bottom: 0px; margin-top: 0px;">oShape.TextFrame.TextRange.Text = j</pre>
        </pre>
    </div>
    <p>Next are the images. This was easy well. I just placed one image manually and checked its properties in the size
        and position box. Took these centimeter values and converted to points and set them as top and left. It all
        there below. The wrapping was important as I wanted it to be square. SoF was also a big help.</p>
    <div style="background: rgb(240, 240, 240); overflow: auto; padding: 0.5em 0.6em; width: auto;">
        <pre style="line-height: 125%; margin: 0px;">file_source = "<i>path_to_file</i>"
img_source = "<i>path_to_image</i>"
    
student_name = c_list.Range("C" &amp; i)
doc_name = student_name &amp; ".docx"
'Make active
Documents.Open (file_source &amp; doc_name)
ActiveDocument.GoTo(What:=wdGoToPage, Count:=1).Select
Set aShape = Selection.InlineShapes.AddPicture(FileName:=img_source, _
    LinkToFile:=False, SaveWithDocument:=True).ConvertToShape
    With aShape
        .WrapFormat.Type = wdWrapTight
        .RelativeHorizontalPosition = wdRelativeHorizontalPositionPage
        .RelativeVerticalPosition = wdRelativeVerticalPositionPage
        .Top = CentimetersToPoints(8.72)
        .Left = CentimetersToPoints(6.94)
        .Select
    End With<br /></pre>
    </div>
    <p>I was almost done at this point. Some basic features like the admission number and name were done through the
        following:</p>
    <div style="background: rgb(240, 240, 240); overflow: auto; padding: 0.5em 0.6em; width: auto;">
        <pre style="line-height: 125%; margin: 0px;">'Set initial and final boxes
Set s = ActiveDocument.Shapes

s(10).TextFrame.TextRange.Text = student_name
s(11).TextFrame.TextRange.Text = "class"
s(12).TextFrame.TextRange.Text = c_list.Range("D" &amp; i + 1).Value 'DOB
s(13).TextFrame.TextRange.Text = "J" 'Section
s(14).TextFrame.TextRange.Text = c_list.Range("B" &amp; i + 1).Value 'Admission number
s(46).TextFrame.TextRange.InsertBefore (c_list.Range("G" &amp; i + 1).Value) 'remarks</pre>
        <pre style="line-height: 125%; margin: 0px;">....</pre>
    </div>
    <p>The final complete Macro has everything just in one Sub. This made debugging much easier.</p>
    <p>Full code:</p>
    <div style="background: rgb(240, 240, 240); overflow: auto; padding: 0.5em 0.6em; width: auto;">
        <pre style="line-height: 125%; margin: 0px;">Sub Fill_all()
'
' Macro1 Macro
'
'
Dim objExcel As Object 
Set objExcel = CreateObject("Excel.Application")
' Get class_list sheet
Set wb = Workbooks.Open("D:\vba_test\source_sheet.xlsx")
Set c_list = wb.Sheets("class_list")

Dim max_students As Integer
Dim student_rating As String

num_students = c_list.UsedRange.Rows.Count - 1

For i = 1 To num_students
    student_name = c_list.Range("C" &amp; i + 1).Value
    student_rating = c_list.Range("E" &amp; i + 1).Value
    
    save_source = "D:\vba_test\"
    doc_name = student_name &amp; ".docx"
    
    'Copy doc
    WordBasic.CopyFileA FileName:=save_source &amp; ActiveDocument.Name, _
    Directory:=save_source &amp; doc_name
    'Make active
    Documents.Open (save_source &amp; doc_name)
    'Set initial and final boxes
    Set s = ActiveDocument.Shapes
    's(8).TextFrame.TextRange.Text = 2020 'academic year from
    's(9).TextFrame.TextRange.Text = 2021 'academic year from
    s(10).TextFrame.TextRange.Text = student_name
    's(11).TextFrame.TextRange.Text = "KG" 'class
    s(12).TextFrame.TextRange.Text = c_list.Range("D" &amp; i + 1).Value 'DOB
    's(13).TextFrame.TextRange.Text = "J" 'Section
    s(14).TextFrame.TextRange.Text = c_list.Range("B" &amp; i + 1).Value 'Admission number
    s(46).TextFrame.TextRange.InsertBefore (c_list.Range("G" &amp; i + 1).Value) 'remarks
    s(47).TextFrame.TextRange.Text = c_list.Range("F" &amp; i + 1).Value 'attendance
    
    Dim max_coms As Integer
Dim j As Integer
Dim oShape As Shape
    If ActiveDocument.Shapes.Count &gt; 0 Then
        For j = 16 To 45 'ActiveDocument.Shapes.Count
            max_coms = WorksheetFunction.CountA(wb.Sheets(student_rating).Rows(j - 15))
            Set oShape = ActiveDocument.Shapes(j)
            If oShape.AutoShapeType = msoShapeRectangle Then 'we need to check both if oShape is of type msoShapeRectangle and its textframe contains place for writing
                If oShape.TextFrame.HasText = True Then
                    oShape.TextFrame.TextRange.Text = wb.Sheets(student_rating).Cells(j - 15, Int(1 + Rnd * (max_coms - 1 + 1))).Value
                    oShape.TextFrame.TextRange.Font.Name = "Arial"
                    oShape.TextFrame.TextRange.ParagraphFormat.LineSpacing = LinesToPoints(1.5)
                    oShape.TextFrame.TextRange.Font.Size = 12
                End If
            End If
        Next
    End If
ActiveDocument.Save
'Uncomment below to save as pdf
'ActiveDocument.ExportAsFixedFormat , OutputFileName:=student_name &amp; ".pdf", _
     ExportFormat:=wdExportFormatPDF
ActiveDocument.Close
Next
End Sub</pre>
    </div>
    <p>There is a cute little save as pdf thingy too.</p>
    <p>It's not as rosy as everything looks though. Since I was doing VBA for the first time, there were a lot of errors
        and problems to debug along the way.:</p>
    <p></p>
    <ol style="text-align: left;">
        <li>One cool problem was that whenever the macro called the excel object as an application it opened it in a
            stream at the backend but never closed it afterwards. This made a lot of background processes during testing
            and to make things worse, when I opened excel GUI, it threw the following read only error " "the_file".xlsx
            is locked for editing by "my_username" user.&nbsp; Open "read only" or click notify...".When this happened,
            any changes in the workbook(GUI) were not reflected as the process from VBA was different than the GUI one.
            This was my mistake of course. To correct it I called the Quit method on the object at the end to close the
            process</li>
        <li>After I gave control over to my mother to fill the excel sheet with the comments, there was another problem.
            It turns out that she ended u filling more comment variations for certain questions than others. The above
            code was written with the assumption that there are exactly 5 variations for every answer. I solved this by
            taking the length of each row separately for randomization:</li>
    </ol>
    <p></p>
    <div style="background: rgb(240, 240, 240); overflow: auto; padding: 0.5em 0.6em; width: auto;">
        <pre style="line-height: 125%; margin: 0px;"><pre style="line-height: 18.7501px; margin-bottom: 0px; margin-top: 0px;"><pre style="line-height: 18.7501px; margin-bottom: 0px; margin-top: 0px;">max_coms = WorksheetFunction.CountA(wb.Sheets(student_rating).Rows(j - 15))
</pre>
        <div>'inside if statement</div>
        <div><span>&nbsp;&nbsp; &nbsp;</span>oShape.TextFrame.TextRange.Text = wb.Sheets(student_rating).Cells(j - 15,
            Int(1 + Rnd * (max_coms - 1 + 1))).Value</div>
        </pre>
        </pre>
    </div>
    <p>And I think that's it! Its not much but it is always super fun to work on a new language + problem. I did this
        last week and it all of it took me about half a day but it was worth it. My mom is happy :)</p>
    <p>In other news, I've made a lot of headway into a reinforcement learning project that I was working on. More
        blogging on the way!</p>
    <p><br /></p>
</div>