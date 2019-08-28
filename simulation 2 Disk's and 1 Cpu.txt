using System;
using System.Collections;
using System.IO;


namespace Performance
{
    class Program
    {
        static void Main(string[] args)
        {
            Random random = new Random();
            int number = 200;//Random number of numbers at files
            String s1 = "E:\\Random_Number1.txt", s2 = "E:\\Random_Number2.txt", 
                s3 = "E:\\Random_Number3.txt", s4 = "E:\\Random_Number4.txt";
            StreamWriter Sw1 = new StreamWriter(s1);
            StreamWriter Sw2 = new StreamWriter(s2);
            StreamWriter Sw3 = new StreamWriter(s3);
            StreamWriter Sw4 = new StreamWriter(s4);
            for (int i = 0; i < number; i++)
            {
                var rand1 = random.Next(100);
                Sw1.WriteLine(rand1);
                var rand2 = random.Next(100);
                Sw2.WriteLine(rand2);
                var rand3 = random.Next(100);
                Sw3.WriteLine(rand3);
                var rand4 = random.Next(1000);
                Sw4.WriteLine(rand4);
            }
            Sw1.Close();
            Sw2.Close();
            Sw3.Close();
            Sw4.Close();
            int Scpunow = 0, Sd1now = 0, Sd2now = 0,
                Scpu = 0, temp = 0, d1d2, d2d1, cpd1d2, mode_arr = 5 , Sum_arr = 0, End = 0,
                ex_old_cpu = 0, 
                wacpu = 0, wad1 = 0, wad2 = 0,  //Waiting time 
                tempwacpu = 0, tempwad1 = 0, tempwad2=0,
                cbusy = 0, d1busy = 0, d2busy = 0,
                equalc = 0, equald1 = 0, equald2 = 0,
                arrival = 0,
                excputime = 0,         //EXIT time to CPU
                exd1time = 0,         //EXIT time to disk1
                exd2time = 0,        //EXIT time to disk2
                arcputime = 0,      //Arrival time to CPU
                ard1time =0,       //Arrival time to Disk1
                ard2time =0,      //Arrival time to Disk2
                Sd1 = 0,         //Disk1 service time
                Sd2 = 0;        //Disk2 service time
            String str1, str2, str3;
            bool flagcp = false, flagd1 = false, flagd2 = false;
            int numreturndisk1, numreturndisk2, //Number of return to disk1 & disk2
                numindisk1, numindisk2;//Number of input to disk1 & disk2
            Console.Title = " FEL Performance ";
            Z: Console.ForegroundColor = ConsoleColor.Green;
            Console.Write("\n\n\t\t 1.default \n\t\t 2.Custom \n\nChoose an option:");
            int select = Convert.ToInt32(Console.ReadLine());
            if (select > 2)
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("\t\t\t\tInvalid Number Please Try Again");
                goto Z;
            }
            Console.Write("Number of Customers:");
            int num = Convert.ToInt32(Console.ReadLine());
            int[] linecpu = new int[num];
            int[] Rand1 = new int[num];
            numindisk1 =Convert.ToInt32( Math.Round (num* 0.3));
            numindisk2 = num - numindisk1;
            numreturndisk1 = Convert.ToInt32(Math.Round(numindisk1 * 0.4));
            numreturndisk2 = Convert.ToInt32(Math.Round(numindisk2 * 0.7));
            Queue cpu = new Queue();       //Cpu Service Random time 
            Queue Servicecpu = new Queue();
            Queue disk1 = new Queue();    //Disk1 Service Random time 
            Queue disk2 = new Queue();   //Disk2 Service Random time 
            Queue excpu = new Queue();  //Exit Random time of Cpu 
            Queue waitcpu = new Queue();
            Queue waitdisk1 = new Queue();
            Queue waitdisk2 = new Queue();
            Queue Sum_arr_now = new Queue();
            ArrayList indisk1 = new ArrayList();// Arrival 30% to Disk1
            ArrayList repeatdisk1 = new ArrayList();
            ArrayList repeatdisk2 = new ArrayList();
            indisk1.Add(random.Next(num) + 1);
            numindisk1 += Convert.ToInt32(numreturndisk1 * 0.3);
            for (int i = 1; i < numindisk1; i++)
            {
                temp = random.Next(num) + 1;
                if (indisk1.Contains(temp) == true)
                    i--;
                else
                {
                    indisk1.Add(temp);
                }
            }
            repeatdisk1.Add(random.Next(numindisk1) + 1);
            for (int i = 1; i < numreturndisk1; i++)
            {
                temp = random.Next(numindisk1) + 1;
                if (repeatdisk1.Contains(temp) == true)
                    i--;
                else
                {
                    repeatdisk1.Add(temp);
                }
            }
            repeatdisk2.Add(random.Next(numindisk2) + 1);
            for (int i = 1; i < numreturndisk2; i++)
            {
                temp = random.Next(numindisk2) + 1;
                if (repeatdisk2.Contains(temp) == true)
                    i--;
                else
                {
                    repeatdisk2.Add(temp);
                }
            }
            StreamReader Sread1 = new StreamReader(s1);
            StreamReader Sread2 = new StreamReader(s2);
            StreamReader Sread3 = new StreamReader(s3);
            StreamReader Sread4 = new StreamReader(s4);
            cpd1d2 = numreturndisk1 + numreturndisk2;
            d1d2 = Convert.ToInt32(cpd1d2 * 0.4);
            d2d1 = cpd1d2 - d1d2;
            if (select == 1)
            {
                for (int i = 0; i < num + cpd1d2; i++)
                {
                    str1 = Sread1.ReadLine();
                    End = Convert.ToInt32(str1);
                    if (End < 25)
                    {
                        End = 1;
                        cpu.Enqueue(End);
                        Servicecpu.Enqueue(End);
                    }
                    else if (End >= 25 && End < 50)
                    {
                        End = 2;
                        cpu.Enqueue(End);
                        Servicecpu.Enqueue(End);
                    }
                    else if (End >= 50 && End < 75)
                    {
                        End = 3;
                        cpu.Enqueue(End);
                        Servicecpu.Enqueue(End);
                    }
                    else
                    {
                        End = 4;
                        cpu.Enqueue(End);
                        Servicecpu.Enqueue(End);
                    }
                    Scpu += End;
                    cbusy += End;
                }
                Sread1.Close();
                for (int i = 0; i < numindisk1 + d1d2; i++)
                {
                    str2 = Sread2.ReadLine();
                    temp = Convert.ToInt16(str2);
                    if (temp < 10)
                        disk1.Enqueue(2);
                    else if (temp >= 10 && temp < 20)
                        disk1.Enqueue(3);
                    else if (temp >= 20 && temp < 40)
                        disk1.Enqueue(4);
                    else
                        disk1.Enqueue(5);
                }
                Sread2.Close();
                for (int i = 0; i < numindisk2 + d2d1; i++)
                {
                    str3 = Sread3.ReadLine();
                    temp = Convert.ToInt32(str3);
                    if (temp < 20)
                        disk2.Enqueue(4);
                    else if (temp >= 20 && temp < 40)
                        disk2.Enqueue(5);
                    else if (temp >= 40 && temp < 80)
                        disk2.Enqueue(6);
                    else
                        disk2.Enqueue(7);
                }
                Sread3.Close();
            }
            else if (select == 2)
            {
                float sumpro = 0, temp1 = 0;
                Console.Write("Number of cpu service time probabilities: ");
                int numprocpu = Convert.ToInt32(Console.ReadLine());
                double[] pro_s_cpu = new double[numprocpu + 1];
                pro_s_cpu[0] = 0;
                Console.WriteLine("Enter {0} numbers in ascending order", numprocpu);
                int[] num_s_cpu = new int[numprocpu];
                for (int i = 0; i < numprocpu; i++)
                {
                    num_s_cpu[i] = Convert.ToInt32(Console.ReadLine());
                }
                Console.WriteLine("Enter for each Number a probability:");
                A: Console.ForegroundColor = ConsoleColor.Green;
                for (int i = 1; i <= numprocpu; i++)
                {
                    Console.Write("Enter  probability for {0}:", num_s_cpu[i - 1]);
                    temp1 = float.Parse(Console.ReadLine());
                    sumpro += temp1;
                    pro_s_cpu[i] = sumpro;
                }
                if (sumpro != 1.0)
                {
                    Console.ForegroundColor = ConsoleColor.Red;
                    Console.WriteLine("probabilities sum not equal to 1 \n\t please Try Again");
                    sumpro = 0;
                    goto A;
                }
                sumpro = 0;
                Console.Write("Number of disk1 service time probabilities: ");
                int numprod1 = Convert.ToInt32(Console.ReadLine());
                double[] pro_s_d1 = new double[numprod1 + 1];
                pro_s_d1[0] = 0;
                Console.WriteLine("Enter {0} numbers in ascending order", numprod1);
                int[] num_s_d1 = new int[numprod1];
                for (int i = 0; i < numprod1; i++)
                {
                    num_s_d1[i] = Convert.ToInt32(Console.ReadLine());
                }
                Console.WriteLine("Enter for each Number a probability:");
                B: Console.ForegroundColor = ConsoleColor.Green;
                for (int i = 1; i <= numprod1; i++)
                {
                    Console.Write("Enter  probability for {0}:", num_s_d1[i - 1]);
                    temp1 = float.Parse(Console.ReadLine());
                    sumpro += temp1;
                    pro_s_d1[i] = sumpro;
                }
                if (sumpro != 1.0)
                {
                    Console.ForegroundColor = ConsoleColor.Red;
                    Console.WriteLine("probabilities sum not equal to 1 \n\t please Try Again");
                    sumpro = 0;
                    goto B;
                }
                sumpro = 0;
                Console.Write("Number of disk2 service time probabilities: ");
                int numprod2 = Convert.ToInt32(Console.ReadLine());
                double[] pro_s_d2 = new double[numprod2 + 1];
                pro_s_d2[0] = 0;
                Console.WriteLine("Enter {0} numbers in ascending order", numprod2);
                int[] num_s_d2 = new int[numprod2];
                for (int i = 0; i < numprod2; i++)
                {
                    num_s_d2[i] = Convert.ToInt32(Console.ReadLine());
                }
                Console.WriteLine("Enter for each Number a probability:");
                C: Console.ForegroundColor = ConsoleColor.Green;
                for (int i = 1; i <= numprod2; i++)
                {
                    Console.Write("Enter  probability for {0}:", num_s_d2[i - 1]);
                    temp1 =float.Parse(Console.ReadLine());
                    sumpro += temp1;
                    pro_s_d2[i] = sumpro;
                }
                if (sumpro != 1)
                {
                    Console.ForegroundColor = ConsoleColor.Red;
                    Console.WriteLine("probabilities sum not equal to 1 \n\t please Try Again");
                    sumpro = 0;
                    goto C;
                }
                sumpro = 0;
                for (int i = 0; i < num + cpd1d2; i++)
                {
                    str1 = Sread1.ReadLine();
                    End = Convert.ToInt32(str1);
                    for (int j = 0; j < numprocpu; j++)
                    {
                        if (End >= pro_s_cpu[j] * 100 && End < pro_s_cpu[j + 1] * 100)
                            End = num_s_cpu[j];
                    }
                    cpu.Enqueue(End);
                    Servicecpu.Enqueue(End);
                }
                Sread1.Close();
                for (int i = 0; i < numindisk1 + d1d2; i++)
                {
                    str2 = Sread2.ReadLine();
                    temp = Convert.ToInt32(str2);
                    for (int j = 0; j < numprod1; j++)
                    {
                        if (temp >= pro_s_d1[j] * 100 && End < pro_s_d1[j + 1] * 100)
                            temp = num_s_d1[j];
                    }
                    disk1.Enqueue(temp);
                }
                Sread2.Close();
                for (int i = 0; i < numindisk2 + d2d1; i++)
                {
                    str3 = Sread3.ReadLine();
                    temp = Convert.ToInt32(str3);
                    for (int j = 0; j < numprod2; j++)
                    {
                        if (temp >= pro_s_d2[j] * 100 && End < pro_s_d2[j + 1] * 100)
                            temp = num_s_d2[j];
                    }
                    disk2.Enqueue(temp);
                }
                Sread3.Close();
            }
            temp = Convert.ToInt32(Sread4.ReadLine());
            temp = temp % mode_arr + 1;
            arrival = temp;
            Sum_arr_now.Enqueue(temp);
            temp = Convert.ToInt32(cpu.Dequeue());
            excpu.Enqueue(temp);
            equalc = temp;
            for (int i = 1; i < num + cpd1d2; i++)
            {
                temp = Convert.ToInt32(Sread4.ReadLine());
                temp = temp % mode_arr + 1;
                arrival += temp;
                if (arrival <= equalc)
                    equalc += Convert.ToInt32(cpu.Dequeue());
                else
                    equalc = arrival + Convert.ToInt32(cpu.Dequeue());
                Sum_arr_now.Enqueue(arrival);
                excpu.Enqueue(equalc);
            }
            int[] id = new int[num + num+1];
            Sread4.Close();
            for (int i = 0; i <= num *2 ; i++)
            {
                id[i] = 100 + i;
            }

            Console.WriteLine("Time  │                              FEL                          │   Waiting time    │   Service Time");
            Console.WriteLine("      │                                                           │ cpu    d1    d2   │  cpu   d1   d2");
            Console.WriteLine("──────┼───────────────────────────────────────────────────────────┼───────────────────┼──────────────────");
            temp = 0;
            Scpu = 0;
            cbusy = 0;
            str1 = "";
            str2 = "──────┼";
            excputime = Convert.ToInt32(excpu.Dequeue());
            int t = 1, k = 0,
                count = 1,       //Arrival number to cpu
                countd1 = 1,    //Arrival number to disk1
                countd2 = 1;   //Arrival number to disk2
            str1 = "0     │"; 
            arcputime = Convert.ToInt32(Sum_arr_now.Dequeue());
            Scpunow += Convert.ToInt32(Servicecpu.Dequeue());
            str1 += " (Ec," + excputime + ") (Ac," + arcputime + ")";//Ac -> Arrival time to cpu , Ec -> Exit time of cpu
            equalc = id[k];
            k++;
            flagcp = true;
            for (int i = str1.Length; i < 66; i++)
                str1 += " ";
            for (int i = str2.Length; i < 66; i++)
                str2 += "─";
            str1 += "│";
            str2 += "┼";
            str1 += "  " + wacpu + "     " + wad1 + "     " +wad2 + "    │";
            str1 += "   " + Scpu + "    " + Sd1 + "    " + Sd2;
            str2 += "───────────────────┼──────────────────";
            Console.WriteLine(str1);
            Console.WriteLine(str2);
            if (arcputime > excputime)
                wacpu = arcputime - excputime;
            tempwacpu = wacpu;
            tempwad1 = excputime;
            tempwad2 = excputime;
            Scpu = Scpunow;
            while (t <= equalc || t <=exd1time || t<=exd2time)
            {
                str1 = "";
                str2 = "──────┼";
                if (t < 10)
                    str1 = t + "     │";
                else if (t < 100)
                    str1 = t + "    │";
                else
                    str1 = t + "   │";
                if (excputime == t && arcputime == t && Sum_arr_now.Count > 0 )
                {
                    equalc = id[count];
                    excputime = Convert.ToInt32(excpu.Dequeue());
                    Scpunow += Convert.ToInt32(Servicecpu.Dequeue());
                    arcputime = Convert.ToInt32(Sum_arr_now.Dequeue());
                    str1 += " (Ec," + excputime + ") (Ac," + arcputime + ")";
                    if (indisk1.Contains(count) == true && flagd1 == true)
                    {
                        waitdisk1.Enqueue(equalc);
                        indisk1.Remove(count);
                    }
                    else if (indisk1.Contains(count) == false && flagd2 == true)
                    {
                        waitdisk2.Enqueue(equalc);
                    }
                    else if (indisk1.Contains(count) == true && flagd1 == false && disk1.Count > 0)
                    {
                        ard1time = t;
                        Sd1 = Convert.ToInt32(disk1.Dequeue());
                        exd1time = t + Sd1;
                        str1 +=" (Ad1,"+ard1time+ ") (Ed1," + exd1time + ")";
                        indisk1.Remove(count);
                        flagd1 = true;
                        equald1 = id[countd1];
                        countd1++;
                    }
                    else if (indisk1.Contains(count) == false && flagd2 == false && disk2.Count > 0)
                    {
                        ard2time = t;
                        Sd2 = Convert.ToInt32(disk1.Dequeue());
                        exd2time = t + Sd2;
                        str1 += " (Ad2," + ard2time + ") (Ed2," + exd2time + ")";
                        flagd2 = true;
                        equald2 = id[countd2];
                        countd2++;
                    }
                    if (waitcpu.Count > 0)
                    {
                        waitcpu.Dequeue();
                    }
                    count++;
                }
                else if (excputime == t && arcputime > t && excpu.Count>0)
                {
                    if (waitcpu.Count == 0)
                    {
                        if (count != 1)
                        {
                            ex_old_cpu = excputime;
                            wacpu += arcputime - ex_old_cpu;
                        }
                        flagcp = false;
                    }
                    else
                    {
                        excputime = Convert.ToInt32(excpu.Dequeue());
                        Scpunow += Convert.ToInt32(Servicecpu.Dequeue());
                        str1 += " (Ec," + excputime + ")";
                        equalc = id[count];
                        waitcpu.Dequeue();
                    }
                    if (indisk1.Contains(count) == true && flagd1 == true)
                    {
                        waitdisk1.Enqueue(equalc);
                        indisk1.Remove(count);
                    }
                    else if (indisk1.Contains(count) == false && flagd2 == true)
                    {
                        waitdisk2.Enqueue(equalc);
                    }
                    else if (indisk1.Contains(count) == true && flagd1 == false && disk1.Count > 0)
                    {
                        wad1 = excputime - tempwad1;
                        ard1time = t;
                        Sd1 = Convert.ToInt32(disk1.Dequeue());
                        exd1time = t + Sd1;
                        str1 += " (Ad1," + ard1time + ") (Ed1," + exd1time + ")";
                        indisk1.Remove(count);
                        flagd1 = true;
                        equald1 = id[countd1];
                        countd1++;
                    }
                    else if (indisk1.Contains(count) == false && flagd2 == false && disk2.Count > 0)
                    {
                        wad2 = excputime - tempwad2;
                        ard2time = t;
                        Sd2 = Convert.ToInt32(disk2.Dequeue());
                        exd2time = t + Sd2;
                        str1 += " (Ad2," + ard2time + ") (Ed2," + exd2time + ")";
                        flagd2 = true;
                        equald2 = id[countd2];
                        countd2++;
                    }
                    count++;
                }
                else if (excputime == t && arcputime < t && excpu.Count > 0)
                {
                    excputime = Convert.ToInt32(excpu.Dequeue());
                    Scpunow += Convert.ToInt32(Servicecpu.Dequeue());
                    str1 += " (Ec," + excputime + ")";
                    if (indisk1.Contains(count) == true && flagd1 == true)
                    {
                        waitdisk1.Enqueue(equalc);
                        indisk1.Remove(count);
                    }
                    else if (indisk1.Contains(count) == false && flagd2 == true)
                    {
                        waitdisk2.Enqueue(equalc);
                    }
                    else if (indisk1.Contains(count) == true && flagd1 == false && disk1.Count > 0)
                    {
                        wad1 = excputime - tempwad1;
                        ard1time = t;
                        Sd1 = Convert.ToInt32(disk1.Dequeue());
                        exd1time = t + Sd1;
                        str1 += " (Ad1," + ard1time + ") (Ed1," + exd1time + ")";//Exit time of disk1
                        indisk1.Remove(count);
                        flagd1 = true;
                        equald1 = id[countd1];
                        countd1++;
                    }
                    else if (indisk1.Contains(count) == false && flagd2 == false && disk2.Count > 0)
                    {
                        wad2 = excputime - tempwad2;
                        ard2time = t;
                        Sd2 = Convert.ToInt32(disk1.Dequeue());
                        exd2time = t + Sd2;
                        str1 += " (Ad2," + ard2time + ") (Ed2," + exd2time + ")";//Exit time of disk2
                        flagd2 = true;
                        equald2 = id[countd2];
                        countd2++;
                    }
                    if (waitcpu.Count > 0)
                    {
                        arcputime = t;
                        waitcpu.Dequeue();
                        count++;
                        equalc = id[count];
                    }
                    else
                        flagcp = false;
                }
                else if (excputime > t && arcputime == t && Sum_arr_now.Count > 0)
                {
                    arcputime = Convert.ToInt32(Sum_arr_now.Dequeue());
                    str1 += " (Ac," + arcputime + ")";
                    waitcpu.Enqueue(id[k]);
                    k++;
                }
                else if (excputime < t && arcputime == t && Sum_arr_now.Count > 0 )
                {
                    if (count != 1)
                    {
                        ex_old_cpu = excputime;
                        wacpu += arcputime - ex_old_cpu;
                    }
                    arcputime = Convert.ToInt32(Sum_arr_now.Dequeue());
                    Scpunow += Convert.ToInt32(Servicecpu.Dequeue());
                    excputime = Convert.ToInt32(excpu.Dequeue());
                    str1 += " (Ec," + excputime + ") (Ac," + arcputime + ")";
                    equalc = id[k];
                    k++;
                    count++;
                    flagcp = true;
                }
                if (exd1time == t)
                {
                    if (repeatdisk1.Contains(countd1) == true && waitcpu.Count > 0)
                    {
                        repeatdisk1.Remove(countd1);
                        equald1 = id[countd1];
                        waitcpu.Enqueue(equald1);
                    }
                    else if (repeatdisk1.Contains(countd1) == true && waitcpu.Count == 0)
                    {
                        repeatdisk1.Remove(countd1);
                        if (flagcp == false)
                        {
                            arcputime = Convert.ToInt32(Sum_arr_now.Dequeue());
                            Scpunow += Convert.ToInt32(Servicecpu.Dequeue());
                            excputime = Convert.ToInt32(excpu.Dequeue());
                            str1 += " (Ec," + excputime + ") (Ac," + arcputime + ")";
                        }
                        equalc = id[k];
                        k++;
                    }
                    if (waitdisk1.Count > 0 && disk1.Count > 0)
                    {
                        ard1time = t;
                        countd1++;
                        equald1 = id[countd1];
                        Sd1 = Convert.ToInt32(disk1.Dequeue());
                        exd1time = t + Sd1;
                        str1 += " (Ad1," + ard1time + ") (Ed1," + exd1time + ")";
                    }
                    else
                        flagd1 = false;
                }
                if (exd2time == t)
                {
                    if (repeatdisk2.Contains(countd2) == true && waitcpu.Count > 0)
                    {
                        repeatdisk2.Remove(countd2);
                        equald2 = id[countd2];
                        waitcpu.Enqueue(equald2);
                    }
                    else if (repeatdisk2.Contains(countd2) == true && waitcpu.Count == 0 && excpu.Count > 0 && Sum_arr_now.Count>0)
                    {
                        repeatdisk2.Remove(countd2);
                        if (flagcp == false)
                        {
                            arcputime = Convert.ToInt32(Sum_arr_now.Dequeue());
                            Scpunow += Convert.ToInt32(Servicecpu.Dequeue());
                            excputime = Convert.ToInt32(excpu.Dequeue());
                            str1 += " (Ec," + excputime + ") (Ac," + arcputime + ")";
                        }
                        equalc = id[k];
                        k++;
                    }
                    if (waitdisk2.Count > 0 && disk2.Count > 0)
                    {
                        ard2time = t;
                        countd2++;
                        equald2 = id[countd2];
                        Sd2 = Convert.ToInt32(disk2.Dequeue());
                        exd2time = t + Sd2;
                        str1 += " (Ad2," + ard2time + ") (Ed2," + exd2time + ")";
                    }
                    else
                        flagd2 = false;
                }
                if ( t == excputime && excpu.Count == 0)
                {
                    if (indisk1.Contains(count) == true && flagd1 == true)
                    {
                        waitdisk1.Enqueue(equalc);
                        indisk1.Remove(count);
                    }
                    else if (indisk1.Contains(count) == false && flagd2 == true)
                    {
                        waitdisk2.Enqueue(equalc);
                    }
                    else if (indisk1.Contains(count) == true && flagd1 == false && disk1.Count > 0)
                    {
                        ard1time = t;
                        Sd1 = Convert.ToInt32(disk1.Dequeue());
                        exd1time = t + Sd1;
                        str1 += " (Ad1," + ard1time + ") (Ed1," + exd1time + ")";
                        indisk1.Remove(count);
                        flagd1 = true;
                        equald1 = id[countd1];
                        countd1++;
                    }
                    else if (indisk1.Contains(count) == false && flagd2 == false && disk2.Count > 0)
                    {
                        ard2time = t;
                        Sd2 = Convert.ToInt32(disk1.Dequeue());
                        exd2time = t + Sd2;
                        str1 += " (Ad2," + ard2time + ") (Ed2," + exd2time + ")";
                        flagd2 = true;
                        equald2 = id[countd2];
                        countd2++;
                    }
                    if (waitcpu.Count > 0)
                    {
                        waitcpu.Dequeue();
                    }
                    equalc = id[count];
                    flagcp = false;
                    count++;
                }
                if (str1.Length > 7)
                {
                    if (exd2time > t && str1.Contains("Ed2") == false)
                        str1 += " (Ed2," + exd2time + ")";
                    if (exd1time > t && str1.Contains("Ed1") == false)
                        str1 += " (Ed1," + exd1time + ")";
                    if (excputime > t && str1.Contains("Ec") == false)
                        str1 += " (Ec," + excputime + ")";
                    if (arcputime > t && str1.Contains("Ac") == false)
                        str1 += " (Ac," + arcputime + ")";
                    for (int i = str1.Length; i < 66; i++)
                        str1 += " ";
                    for (int i = str2.Length; i < 66; i++)
                        str2 += "─";
                    str1 += "│";
                    str2 += "┼";
                    if (tempwacpu < 10)
                        str1 +="  " + tempwacpu + "     ";
                    else if (tempwacpu < 100)
                        str1 +=" " + tempwacpu  + "     ";
                    else
                        str1 +=" " + tempwacpu +"    ";
                    if (tempwad1 < 10)
                        str1 += tempwad1 + "     ";
                    else if (tempwad1 < 100)
                        str1 += tempwad1 + "    ";
                    else
                        str1 += tempwad1 + "   ";
                    if (tempwad2 < 10)
                        str1 += tempwad2 + "    │";
                    else if (tempwad2 <100)
                        str1 += tempwad2 + "   │";
                    else
                        str1 += tempwad2 + "  │";
                    if (Scpu < 10)
                        str1 += "   " + Scpu+ "    ";
                    else if (Scpu < 100)
                        str1 += "  " + Scpu + "    ";
                    else
                        str1 += "  " + Scpu  +"  ";
                    if (Sd1now < 10)
                        str1 += Sd1now + "    ";
                    else if (Sd1now < 100)
                        str1 += Sd1now + "    ";
                    else
                        str1 += Sd1now + "   ";
                    if (Sd2now < 10)
                        str1 += Sd2now ;
                    else if ( Sd2now < 100)
                        str1 += Sd2now ;
                    else
                        str1 += Sd2now ;
                    str2 += "───────────────────┼──────────────────";
                    Console.WriteLine(str1);
                    Console.WriteLine(str2);
                    tempwacpu = wacpu;
                    tempwad1 += wad1;
                    tempwad2 += wad2;
                    wad2 = 0;
                    wad1 = 0;
                    Scpu = Scpunow;
                    Sd1now += Sd1;
                    Sd2now += Sd2;
                    Sd1 = 0;
                    Sd2 = 0;
                }
                t++;
            }
            double Xcpu ,Xd1,Xd2 , Ucpu ,Ud1 , Ud2;
            temp=Math.Max(exd1time, exd2time);//temp -> T
            Ucpu = Convert.ToDouble(Scpu) / Convert.ToDouble(temp);
            Ud1 = Convert.ToDouble(Sd1now) / Convert.ToDouble(temp);
            Ud2 = Convert.ToDouble(Sd2now) / Convert.ToDouble(temp);
            Xcpu = Convert.ToDouble(num + cpd1d2) / Convert.ToDouble(temp);
            Xd1 = Convert.ToDouble(countd1) / Convert.ToDouble(temp);
            Xd2 = Convert.ToDouble(countd2) / Convert.ToDouble(temp);
            Console.WriteLine("\n\n\t\t Cpu busy Time : {0} ", Scpu);
            Console.WriteLine("\n\n\t\t Disk1 busy Time : {0} ", Sd1now);
            Console.WriteLine("\n\n\t\t Disk2 busy Time : {0} ", Sd2now);
            Console.WriteLine("\n\n\t\t Last Exit Time of Cpu : {0} ", excputime);
            Console.WriteLine("\n\n\t\t Last Exit Time of Disk1 : {0} ", exd1time);
            Console.WriteLine("\n\n\t\t Last Exit Time of Disk2 : {0} ", exd2time);
            Console.WriteLine("\n\n\t\t CPU Trothput : {0} ", Xcpu);
            Console.WriteLine("\n\n\t\t Disk1 Trothput : {0} ", Xd1);
            Console.WriteLine("\n\n\t\t Disk2 Trothput : {0} ", Xd2);
            Console.WriteLine("\n\n\t\t CPU Utilization : {0} ", Ucpu);
            Console.WriteLine("\n\n\t\t Disk1 Utilization : {0} ", Ud1);
            Console.WriteLine("\n\n\t\t Disk2 Utilization : {0} ", Ud2);
            Console.ReadKey();
        }
    }
}
