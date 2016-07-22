---
title: Java常见IO流的使用
date: 2016-01-14
author:  KevinWu
categories: Java
tags: 
	- Java 
	- IO
---
都在复习Java期末考试，貌似大家最害怕的就是这个（估计期末考试难度也就考到这个了），所以就总结一下写出来了供参考：
直接上代码：
<!--more-->

``` java
import java.io.*;
/**
 * 
 * @author KevinWu 文件操作示例
 *
 */
public class FileHandle {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		FileHandle myFileHandle = new FileHandle();
		myFileHandle.readTxtFile("a.txt");
		myFileHandle.copyTxtFile("a.txt", "b.txt");
		myFileHandle.copyNonTxtFile("111.pdf", "222.pdf");
		myFileHandle.outPut("测试输出字符串", "test.txt");
		myFileHandle.creatFolder(".", "我是新建的文件夹");// 路径给点，表示当前目录
	}

	/**
	 * 方法作用：读取文本文件的方法 参数：待读取文件的路径
	 */
	public void readTxtFile(String file_dir) {
		// 创建并实例化文件对象
		File myFile = new File(file_dir);
		// 判断该对象对应的路径下是不是存在这个文件和是不是文件类型
		if (!myFile.exists() || !myFile.isFile()) {
			System.out.println("文件不存在或不是文件类型");
		} else {
			// 新建FileReader对象
			FileReader myFileReader = null;
			// 新建BufferedReader对象
			BufferedReader myBufferedReader = null;
			try {
				// 实例化FileReader对象
				myFileReader = new FileReader(myFile);
				// 实例化BufferedReader对象
				myBufferedReader = new BufferedReader(myFileReader);
				// 定义一个字符串来接收文件的一行内容
				String str = "";
				while ((str = myBufferedReader.readLine()) != null) {
					// 把读取到的内容输出
					System.out.println(str);
				}
			} catch (IOException e) {
				e.printStackTrace();
			} finally {
				// 在这里关闭缓冲流和输入流，注意要一层一层关
				if (myBufferedReader != null) {
					try {
						myBufferedReader.close();
					} catch (IOException e) {
						e.printStackTrace();
					}
				}
				if (myFileReader != null) {
					try {
						myFileReader.close();
					} catch (IOException e) {
						e.printStackTrace();
					}
				}
			}
		}
	}

	/**
	 * 方法作用：复制文本文件方法 参数1：待复制的文件的路径 参数2：复制目标的文件的路径
	 */
	public void copyTxtFile(String f_dir, String t_dir) {
		// 创建并实例化文件对象
		File fFile = new File(f_dir);
		File tFile = new File(t_dir);
		// 先判断源文件存不存在和是不是文件类型，就不考虑文件是否可读的问题了
		if (!fFile.exists() || !fFile.isFile()) {
			System.out.println("文件不存在或者不是文件类型");
		} else {
			// 创建FileReader对象
			FileReader myFileReader = null;
			// 创建FileWriter对象
			FileWriter myFileWriter = null;
			// 创建BufferedReader和BufferedWriter对象
			BufferedReader myBufferedReader = null;
			BufferedWriter myBufferedWriter = null;
			try {
				// 实例化FileReader和BufferedReader对象
				myFileReader = new FileReader(fFile);
				myBufferedReader = new BufferedReader(myFileReader);
				// 实例化FileWriter和BufferedWriter对象
				myFileWriter = new FileWriter(tFile);
				myBufferedWriter = new BufferedWriter(myFileWriter);
				// 用一个字符串接受数据
				String str = "";
				while ((str = myBufferedReader.readLine()) != null) {
					myBufferedWriter.write(str);
					// 换行
					myBufferedWriter.newLine();
					// 刷新缓冲流
					myBufferedWriter.flush();
				}
			} catch (IOException e) {
				e.printStackTrace();
			} finally {
				if (myBufferedWriter != null) {
					try {
						myBufferedWriter.close();
					} catch (IOException e) {
						e.printStackTrace();
					}
				}
				if (myFileWriter != null) {
					try {
						myFileWriter.close();
					} catch (IOException e) {
						e.printStackTrace();
					}
				}
				if (myBufferedReader != null) {
					try {
						myBufferedReader.close();
					} catch (IOException e) {
						e.printStackTrace();
					}
				}
				if (myFileReader != null) {
					try {
						myFileReader.close();
					} catch (IOException e) {
						e.printStackTrace();
					}
				}
			}
		}
	}

	/**
	 * 方法作用：复制非文本文件方法 参数1：待复制文件的路径 参数2：复制目标文件的路径
	 */
	public void copyNonTxtFile(String f_dir, String t_dir) {
		File fFile = new File(f_dir);
		File dFile = new File(t_dir);
		if (!fFile.exists() || !fFile.isFile()) {
			System.out.println("文件.....");
		} else {
			FileInputStream fis = null;
			FileOutputStream fos = null;
			BufferedInputStream bis = null;
			BufferedOutputStream bos = null;
			try {
				fis = new FileInputStream(fFile);
				fos = new FileOutputStream(dFile);
				bis = new BufferedInputStream(fis);
				bos = new BufferedOutputStream(fos);
				byte b[] = new byte[1024];
				int length;
				while ((length = bis.read(b)) != -1) {
					bos.write(b, 0, length);
					bos.flush();
				}
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			} finally {
				if (bos != null) {
					try {
						bos.close();
					} catch (IOException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
				if (fos != null) {
					try {
						fos.close();
					} catch (IOException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
				if (bis != null) {
					try {
						bis.close();
					} catch (IOException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
				if (fis != null) {
					try {
						fis.close();
					} catch (IOException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
			}

		}
	}

	/**
	 * 方法作用：把字符串内容输出到文本文件 参数1：待输出的字符串 参数2：目标文件路径
	 */
	public void outPut(String dstStr, String f_dir) {
		File fFile = new File(f_dir);
		try {
			FileWriter myFileWriter = new FileWriter(fFile, true);// 设置为追加
			PrintWriter pw = new PrintWriter(myFileWriter);
			pw.println(dstStr);
			// 方便起见直接在这里关闭了
			pw.close();
			myFileWriter.close();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

	/**
	 * 创建文件夹 参数1：待创建的文件夹的路径 参数2：待创建的文件夹的名称
	 */
	public void creatFolder(String f_dir, String f_name) {
		File fDir = new File(f_dir + "/" + f_name);
		if (fDir.exists()) {
			System.out.println("文件夹已经存在");
		} else {
			fDir.mkdirs();// 创建文件夹，如果父目录不存在一并创建
		}
	}

}
```
