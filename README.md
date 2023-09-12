# Image-to-pdf-project

import 'dart:io';

import 'package:another_flushbar/flushbar.dart';
import 'package:flutter/material.dart';
import 'package:image_picker/image_picker.dart';
import 'package:path_provider/path_provider.dart';
import 'package:pdf/pdf.dart';
import 'package:pdf/widgets.dart' as pw;

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  final picker = ImagePicker();
  final pdf = pw.Document();
  File? image;
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          getImage();
        },
        child: Icon(Icons.add),
      ),
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: Text("Image to pdf"),
        actions: [
          IconButton(
              onPressed: () {
                createpdf();
                savepdf();
                setState(() {
                  image = null;
                });
              },
              icon: Icon(Icons.download))
        ],
      ),
      body: image != null
          ? Container(
              height: 400,
              padding: EdgeInsets.all(10),
              decoration:
                  BoxDecoration(borderRadius: BorderRadius.circular(20)),
              width: double.infinity,
              child: Image.file(
                image!.absolute,
                fit: BoxFit.cover,
              ),
            )
          : Container(),
    );
  }

  Future getImage() async {
    final pickedFile = await picker.pickImage(source: ImageSource.gallery);
    setState(() {
      if (pickedFile != null) {
        image = File(pickedFile.path);
      } else {
        print('no image select');
      }
    });
  }

  createpdf() async {
    final _image = pw.MemoryImage(image!.readAsBytesSync());
    pdf.addPage(pw.Page(
        pageFormat: PdfPageFormat.a4,
        build: (pw.Context context) {
          return pw.Center(
            child: pw.Image(_image),
          );
        }));
  }

  savepdf() async {
    try {
      var id = DateTime.now().millisecond;
      final dir = await getExternalStorageDirectory();
      final file = File('${dir!.path}/$id+filename.pdf');
      await file.writeAsBytes(await pdf.save());
      meassage('Successful', 'save to the document');
    } catch (e) {
      meassage('error', e.toString());
    }
  }

  meassage(String title, String msg) {
    Flushbar(
      title: title,
      message: msg,
      duration: Duration(seconds: 3),
      icon: Icon(Icons.downhill_skiing),
    )..show(context);
  }
}
