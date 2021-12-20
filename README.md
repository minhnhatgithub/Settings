 void Delay(int timedelay)
        {
            while (timedelay > 0)
            {
                Thread.Sleep(TimeSpan.FromSeconds(1));
                timedelay--;
                if (isStop)
                    break;
            }
        }
        bool edit_row(int rows, int cell, string value, bool showtime = false, int time = 10)
        {
            try
            {
                if (showtime)
                {
                    while (time > 0)
                    {
                        Delay(1);
                        DataGridViewRow row = dataGridView1.Rows[rows];
                        row.Cells[cell].Value = value + time + " giây";
                        time--;
                    }
                    return true;
                }
                else
                {
                    DataGridViewRow row = dataGridView1.Rows[rows];
                    row.Cells[cell].Value = value;
                    return true;
                }

            }
            catch (Exception)
            {
                return false;
            }
        }
        private static string RunCMD(string cmd)
        {
            Process process = new Process();
            process.StartInfo.FileName = "cmd.exe";
            process.StartInfo.Arguments = "/c " + cmd;
            process.StartInfo.RedirectStandardOutput = true;
            process.StartInfo.UseShellExecute = false;
            process.StartInfo.CreateNoWindow = true;
            process.Start();
            string text = process.StandardOutput.ReadToEnd();
            process.WaitForExit();
            bool flag = string.IsNullOrEmpty(text);
            string result;
            if (flag)
            {
                result = "";
            }
            else
            {
                result = text;
            }
            return result;
        }
        public static string DumpText(string deviceID, string button, int exitwhile, bool dumx = false)
        {
            int num = 0;
            while (true)
            {
                if (dumx == false)
                {
                    RunCMD("adb -s " + deviceID + " shell uiautomator dump /sdcard/window_dump_" + deviceID + ".xml && adb -s " + deviceID + " pull /sdcard/window_dump_" + deviceID + ".xml");
                }
                XmlDocument doc = new XmlDocument();
                doc.Load("window_dump_" + deviceID + ".xml");
                XmlNode book;
                XmlNode root = doc.DocumentElement;

                book = root.SelectSingleNode("//node[@text=\'" + button + "\']");
                if (book != null)
                {
                    string toadonext = book.Attributes["bounds"].Value;
                    toadonext = toadonext.Replace("][", ",").Replace("]", "").Replace("[", "");
                    string toado_1 = toadonext.Split(',')[0];
                    string toado_2 = toadonext.Split(',')[1];
                    int toado1 = Int32.Parse(toado_1);
                    int toado2 = Int32.Parse(toado_2);
                    KAutoHelper.ADBHelper.Tap(deviceID, toado1, toado2);

                    return "0";
                }
                else
                {
                    num++;
                    if (num == exitwhile)
                    {
                        return "1";
                    }
                }
            }
        }
        public static void InputUnicode(string deviceID, string textpull, bool slow = false)
        {
            RunCMD("adb -s " + deviceID + " shell ime set com.android.adbkeyboard/.AdbIME");
            if (slow)
            {
                char[] ArrInputText = textpull.ToCharArray();
                foreach (var ch in ArrInputText)
                {
                    RunCMD("adb -s " + deviceID + " shell am broadcast -a ADB_INPUT_B64 --es msg \'" + Convert.ToBase64String(Encoding.UTF8.GetBytes(ch.ToString())) + "\'");
                }
            }
            else
            {
                string text = Convert.ToBase64String(Encoding.UTF8.GetBytes(textpull));
                RunCMD("adb -s " + deviceID + " shell am broadcast -a ADB_INPUT_B64 --es msg " + text);
            }
        }
        public static void textdump(string deviceID, string click, bool tru = false, int socantru = 1000)
        {

            RunCMD("adb -s " + deviceID + " shell uiautomator dump /sdcard/window_dump_" + deviceID + ".xml && adb -s " + deviceID + " pull /sdcard/window_dump_" + deviceID + ".xml");
            MemoryStream memoryStream = new MemoryStream();
            XmlTextWriter xmlTextWriter = new XmlTextWriter(memoryStream, Encoding.Unicode);
            XmlDocument xmlDocument = new XmlDocument();
            xmlDocument.Load("window_dump_" + deviceID + ".xml");
            xmlTextWriter.Formatting = Formatting.Indented;
            xmlDocument.WriteContentTo(xmlTextWriter);
            xmlTextWriter.Flush();
            memoryStream.Flush();
            memoryStream.Position = 0L;
            StreamReader streamReader = new StreamReader(memoryStream);
            string text = streamReader.ReadToEnd();
            memoryStream.Close();
            xmlTextWriter.Close();
            text = text.Replace(" ", "");
            text = text.Replace("=", "");
            text = text.Replace("\"", "");
            text = text.Replace("-", "");
            text = text.Replace("(", "");
            text = text.Replace(")", "");
            text = text.Replace("/", "");
            text = text.Replace("[", "XXX");
            text = text.Replace("]", "YYY");
            string inpu = Regex.Match(text, click + "[a-z,A-Z,0-9,:,.,_]{0,}").ToString();
            string text1 = Regex.Match(inpu, "XXX[a-z,A-Z,0-9,:,.,_]{0,}").ToString();
            text1 = text1.Replace("XXX", "");
            text1 = text1.Replace("YYY", ",");
            if (text1 != inpu)
            {
                text1 = text1.Replace("][", ",").Replace("]", "").Replace("[", "");
                string toado_1 = text1.Split(',')[0];
                string toado_2 = text1.Split(',')[1];
                int toado1 = Int32.Parse(toado_1);
                int toado2 = Int32.Parse(toado_2);
                if (tru)
                {
                    KAutoHelper.ADBHelper.Tap(deviceID, toado1, toado2 - socantru);
                }
                else
                {
                    KAutoHelper.ADBHelper.Tap(deviceID, toado1, toado2);
                }
            }
            else
            {
                return;
            }

        }
        public void ChangerImei(string deviceID, int rows)
        {
            edit_row(rows, 5, "Mở app com.device.emulator.pro");
            RunCMD("adb -s " + deviceID + " shell monkey -p com.device.emulator.pro -c android.intent.category.LAUNCHER 1");
            edit_row(rows, 5, "Chọn ngẫu nhiên Imei");
            int lapne = 2;
            while (true)
            {
                string click = "Randomall";
                RunCMD("adb -s " + deviceID + " shell uiautomator dump /sdcard/window_dump_" + deviceID + ".xml && adb -s " + deviceID + " pull /sdcard/window_dump_" + deviceID + ".xml");
                MemoryStream memoryStream = new MemoryStream();
                XmlTextWriter xmlTextWriter = new XmlTextWriter(memoryStream, Encoding.Unicode);
                XmlDocument xmlDocument = new XmlDocument();
                xmlDocument.Load("window_dump_" + deviceID + ".xml");
                xmlTextWriter.Formatting = Formatting.Indented;
                xmlDocument.WriteContentTo(xmlTextWriter);
                xmlTextWriter.Flush();
                memoryStream.Flush();
                memoryStream.Position = 0L;
                StreamReader streamReader = new StreamReader(memoryStream);
                string text = streamReader.ReadToEnd();
                memoryStream.Close();
                xmlTextWriter.Close();
                text = text.Replace(" ", "");
                text = text.Replace("=", "");
                text = text.Replace("\"", "");
                text = text.Replace("-", "");
                text = text.Replace("(", "");
                text = text.Replace(")", "");
                text = text.Replace("/", "");
                text = text.Replace("[", "XXX");
                text = text.Replace("]", "YYY");
                string inpu = Regex.Match(text, click + "[a-z,A-Z,0-9,:,.,_]{0,}").ToString();
                string text1 = Regex.Match(inpu, "XXX[a-z,A-Z,0-9,:,.,_]{0,}").ToString();
                text1 = text1.Replace("XXX", "");
                text1 = text1.Replace("YYY", ",");
                if (text1 != inpu)
                {
                    text1 = text1.Replace("][", ",").Replace("]", "").Replace("[", "");
                    string toado_1 = text1.Split(',')[0];
                    string toado_2 = text1.Split(',')[1];
                    int toado1 = Int32.Parse(toado_1);
                    int toado2 = Int32.Parse(toado_2);
                    KAutoHelper.ADBHelper.Tap(deviceID, toado1, toado2);
                }
                else
                {
                }
                lapne--;
                while (lapne <= 0)
                {
                    goto thoat;
                }
            }
            thoat:;
            edit_row(rows, 5, "Chọn fast reboot");
            int lapne1 = 2;
            while (true)
            {
                string click = "FastReboot";
                RunCMD("adb -s " + deviceID + " shell uiautomator dump /sdcard/window_dump_" + deviceID + ".xml && adb -s " + deviceID + " pull /sdcard/window_dump_" + deviceID + ".xml");
                MemoryStream memoryStream = new MemoryStream();
                XmlTextWriter xmlTextWriter = new XmlTextWriter(memoryStream, Encoding.Unicode);
                XmlDocument xmlDocument = new XmlDocument();
                xmlDocument.Load("window_dump_" + deviceID + ".xml");
                xmlTextWriter.Formatting = Formatting.Indented;
                xmlDocument.WriteContentTo(xmlTextWriter);
                xmlTextWriter.Flush();
                memoryStream.Flush();
                memoryStream.Position = 0L;
                StreamReader streamReader = new StreamReader(memoryStream);
                string text = streamReader.ReadToEnd();
                memoryStream.Close();
                xmlTextWriter.Close();
                text = text.Replace(" ", "");
                text = text.Replace("=", "");
                text = text.Replace("\"", "");
                text = text.Replace("-", "");
                text = text.Replace("(", "");
                text = text.Replace(")", "");
                text = text.Replace("/", "");
                text = text.Replace("[", "XXX");
                text = text.Replace("]", "YYY");
                string inpu = Regex.Match(text, click + "[a-z,A-Z,0-9,:,.,_]{0,}").ToString();
                string text1 = Regex.Match(inpu, "XXX[a-z,A-Z,0-9,:,.,_]{0,}").ToString();
                text1 = text1.Replace("XXX", "");
                text1 = text1.Replace("YYY", ",");
                if (text1 != inpu)
                {
                    text1 = text1.Replace("][", ",").Replace("]", "").Replace("[", "");
                    string toado_1 = text1.Split(',')[0];
                    string toado_2 = text1.Split(',')[1];
                    int toado1 = Int32.Parse(toado_1);
                    int toado2 = Int32.Parse(toado_2);
                    KAutoHelper.ADBHelper.Tap(deviceID, toado1, toado2);
                }
                else
                {
                }
                lapne1--;
                while (lapne1 <= 0)
                {
                    goto thoat1;
                }
            }
            thoat1:;
            Delay(10);
            edit_row(rows, 5, "Click Ok");
            int lapne2 = 2;
            while (true)
            {
                string click = "OK";
                RunCMD("adb -s " + deviceID + " shell uiautomator dump /sdcard/window_dump_" + deviceID + ".xml && adb -s " + deviceID + " pull /sdcard/window_dump_" + deviceID + ".xml");
                MemoryStream memoryStream = new MemoryStream();
                XmlTextWriter xmlTextWriter = new XmlTextWriter(memoryStream, Encoding.Unicode);
                XmlDocument xmlDocument = new XmlDocument();
                xmlDocument.Load("window_dump_" + deviceID + ".xml");
                xmlTextWriter.Formatting = Formatting.Indented;
                xmlDocument.WriteContentTo(xmlTextWriter);
                xmlTextWriter.Flush();
                memoryStream.Flush();
                memoryStream.Position = 0L;
                StreamReader streamReader = new StreamReader(memoryStream);
                string text = streamReader.ReadToEnd();
                memoryStream.Close();
                xmlTextWriter.Close();
                text = text.Replace(" ", "");
                text = text.Replace("=", "");
                text = text.Replace("\"", "");
                text = text.Replace("-", "");
                text = text.Replace("(", "");
                text = text.Replace(")", "");
                text = text.Replace("/", "");
                text = text.Replace("[", "XXX");
                text = text.Replace("]", "YYY");
                string inpu = Regex.Match(text, click + "[a-z,A-Z,0-9,:,.,_]{0,}").ToString();
                string text1 = Regex.Match(inpu, "XXX[a-z,A-Z,0-9,:,.,_]{0,}").ToString();
                text1 = text1.Replace("XXX", "");
                text1 = text1.Replace("YYY", ",");
                if (text1 != inpu)
                {
                    text1 = text1.Replace("][", ",").Replace("]", "").Replace("[", "");
                    string toado_1 = text1.Split(',')[0];
                    string toado_2 = text1.Split(',')[1];
                    int toado1 = Int32.Parse(toado_1);
                    int toado2 = Int32.Parse(toado_2);
                    KAutoHelper.ADBHelper.Tap(deviceID, toado1, toado2);
                }
                else
                {

                }
                lapne2--;
                while (lapne2 <= 0)
                {
                    goto thoat2;
                }
            }
            thoat2:;
            edit_row(rows, 6, "Hoàn thành fake Imei");
            KAutoHelper.ADBHelper.Key(deviceID, KAutoHelper.ADBKeyEvent.KEYCODE_BACK);
        }
        public string GetXml(string deviceID)
        {
            string aa = "";
            try
            {
                RunCMD("adb -s " + deviceID + " shell uiautomator dump /sdcard/window_dump_" + deviceID + ".xml && adb -s " + deviceID + " pull /sdcard/window_dump_" + deviceID + ".xml");
                MemoryStream memoryStream = new MemoryStream();
                XmlTextWriter xmlTextWriter = new XmlTextWriter(memoryStream, Encoding.Unicode);
                XmlDocument xmlDocument = new XmlDocument();
                xmlDocument.Load("window_dump_" + deviceID + ".xml");
                xmlTextWriter.Formatting = Formatting.Indented;
                xmlDocument.WriteContentTo(xmlTextWriter);
                xmlTextWriter.Flush();
                memoryStream.Flush();
                memoryStream.Position = 0L;
                StreamReader streamReader = new StreamReader(memoryStream);
                string text = streamReader.ReadToEnd();
                memoryStream.Close();
                xmlTextWriter.Close();
                text = text.Replace(" ", "");
                text = text.Replace("=", "");
                text = text.Replace("\"", "");
                text = text.Replace("-", "");
                text = text.Replace("(", "");
                text = text.Replace(")", "");
                text = text.Replace("/", "");
                text = text.Replace("[", "XXX");
                aa = text.Replace("]", "YYY");
            }
            catch
            {

            }
            return aa;
        }
        public static void ClickText(string deviceID, string text, string button, bool tru, int socantru)
        {
            try
            {
                string inpu = Regex.Match(text, button + "[a-z,A-Z,0-9,:,.,_]{0,}").ToString();
                string text1 = Regex.Match(inpu, "XXX[a-z,A-Z,0-9,:,.,_]{0,}").ToString();
                text1 = text1.Replace("XXX", "");
                text1 = text1.Replace("YYY", ",");
                if (text1 != inpu)
                {
                    text1 = text1.Replace("][", ",").Replace("]", "").Replace("[", "");
                    string toado_1 = text1.Split(',')[0];
                    string toado_2 = text1.Split(',')[1];
                    int toado1 = Int32.Parse(toado_1);
                    int toado2 = Int32.Parse(toado_2);
                    if (tru)
                    {
                        KAutoHelper.ADBHelper.Tap(deviceID, toado1, toado2 - socantru);
                    }
                    else
                    {
                        KAutoHelper.ADBHelper.Tap(deviceID, toado1, toado2);
                    }
                }
            }
            catch
            {

            }
        }
