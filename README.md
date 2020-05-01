# SoalShiftSISOP20_modul4_T07

Di suatu perusahaan, terdapat pekerja baru yang super jenius, ia bernama jasir. Jasir baru bekerja selama seminggu di perusahan itu, dalam waktu seminggu tersebut ia selalu terhantui oleh ketidak amanan dan ketidak efisienan file system yang digunakan perusahaan tersebut. Sehingga ia merancang sebuah file system yang sangat aman dan efisien.

Berikut adalah detail filesystem rancangan jasir:
1. Enkripsi versi 1:
  -Jika sebuah direktori dibuat dengan awalan “encv1_”, maka direktori tersebut akan menjadi direktori terenkripsi menggunakan metode        enkripsi v1.
  
  - Jika sebuah direktori di-rename dengan awalan “encv1_”, maka direktori tersebut akan menjadi direktori terenkripsi menggunakan metode enkripsi v1.
  
  - Apabila sebuah direktori terenkripsi di-rename menjadi tidak terenkripsi, maka isi adirektori tersebut akan terdekrip.
  
  - Setiap pembuatan direktori terenkripsi baru (mkdir ataupun rename) akan tercatat ke sebuah database/log berupa file.
  
  - Semua file yang berada dalam direktori ter enkripsi menggunakan caesar cipher dengan key.
    
    9(ku@AW1[Lmvgax6q`5Y2Ry?+sF!^HKQiBXCUSe&0M.b%rI'7d)o4~VfZ*{#:}ETt$3J-zpc]lnh8,GwP_ND|jO

    Misal kan ada file bernama “kelincilucu.jpg” dalam directory FOTO_PENTING, dan key yang dipakai adalah 10
    “encv1_rahasia/FOTO_PENTING/kelincilucu.jpg” => “encv1_rahasia/ULlL@u]AlZA(/g7D.|_.Da_a.jpg
    
  Note : Dalam penamaan file ‘/’ diabaikan, dan ekstensi tidak perlu di encrypt.
  
Metode enkripsi pada suatu direktori juga berlaku kedalam direktori lainnya yang ada didalamnya.


2. Enkripsi versi 2:
- Jika sebuah direktori dibuat dengan awalan “encv2_”, maka direktori tersebut akan menjadi direktori terenkripsi menggunakan metode enkripsi v2.

- Jika sebuah direktori di-rename dengan awalan “encv2_”, maka direktori tersebut akan menjadi direktori terenkripsi menggunakan metode enkripsi v2.

  - Apabila sebuah direktori terenkripsi di-rename menjadi tidak terenkripsi, maka isi direktori tersebut akan terdekrip.
  
  - Setiap pembuatan direktori terenkripsi baru (mkdir ataupun rename) akan tercatat ke sebuah database/log berupa file.
  
  - Pada enkripsi v2, file-file pada direktori asli akan menjadi bagian-bagian kecil sebesar 1024 bytes dan menjadi normal ketika diakses melalui filesystem rancangan jasir. 
  Sebagai contoh, file File_Contoh.txt berukuran 5 kB pada direktori asli akan menjadi 5 file kecil yakni: File_Contoh.txt.000, File_Contoh.txt.001, File_Contoh.txt.002, File_Contoh.txt.003, dan File_Contoh.txt.004.
  
  - Metode enkripsi pada suatu direktori juga berlaku kedalam direktori lain yang ada didalam direktori tersebut (rekursif).

3. Sinkronisasi direktori otomatis:

Tanpa mengurangi keumuman, misalkan suatu directory bernama dir akan tersinkronisasi dengan directory yang memiliki nama yang sama dengan awalan sync_ yaitu sync_dir. Persyaratan untuk sinkronisasi yaitu:
  - Kedua directory memiliki parent directory yang sama.
  - Kedua directory kosong atau memiliki isi yang sama. Dua directory dapat dikatakan memiliki isi yang sama jika memenuhi:
    a. Nama dari setiap berkas di dalamnya sama.
    
    b. Modified time dari setiap berkas di dalamnya tidak berselisih lebih dari 0.1 detik.
  - Sinkronisasi dilakukan ke seluruh isi dari kedua directory tersebut, tidak hanya di satu child directory saja.
  - Sinkronisasi mencakup pembuatan berkas/directory, penghapusan berkas/directory, dan pengubahan berkas/directory.
  
  
Jika persyaratan di atas terlanggar, maka kedua directory tersebut tidak akan tersinkronisasi lagi.
Implementasi dilarang menggunakan symbolic links dan thread.

4. Log system:

  - Sebuah berkas nantinya akan terbentuk bernama "fs.log" di direktori *home* pengguna (/home/[user]/fs.log) yang berguna menyimpan daftar perintah system call yang telah dijalankan.
  - Agar nantinya pencatatan lebih rapi dan terstruktur, log akan dibagi menjadi beberapa level yaitu INFO dan WARNING.
Untuk log level WARNING, merupakan pencatatan log untuk syscall rmdir dan unlink.
  - Sisanya, akan dicatat dengan level INFO.
  
 
  format : [LEVEL]::[yy][mm][dd]-[HH]:[MM]:[SS]::[CMD]::[DESC ...]
  
- LEVEL    : Level logging
- yy   	   :  Tahun dua digit
- mm    	 : Bulan dua digit
- dd    	 : Hari dua digit
- HH    	 : Jam dua digit
- MM    	 : Menit dua digit
- SS    	 : Detik dua digit
- CMD      : System call yang terpanggil
- DESC     : Deskripsi tambahan (bisa lebih dari satu, dipisahkan dengan ::)



Contoh format logging nantinya seperti:

INFO::200419-18:29:28::MKDIR::/iz1

INFO::200419-18:29:33::CREAT::/iz1/yena.jpg

INFO::200419-18:29:33::RENAME::/iz1/yena.jpg::/iz1/yena.jpeg


## JAWABAN 

List library yang akan digunakan
      #define FUSE_USE_VERSION 28
      #include <sys/time.h>
      #include <fuse.h>
      #include <sys/types.h>
      #include <stdio.h>
      #include <pthread.h>
      #include <string.h>
      #include <time.h>
      #include <unistd.h>
      #include <fcntl.h>
      #include <grp.h>
      #include <dirent.h>
      #include <pwd.h>
      #include <errno.h>

    static const char *dirpath = "/home/nabella/Documents";  

Fungsi untuk decrypt
cara kerjanya ialah dengan mengecek di posisi manakah huruf teks[i] yang telah terenkripsi pada huruf caesar, kemudian dikembalikan lagi dengan cara mengubahnya dengan huruf caesar yang berada pada posisi j-10

    char* Decrypt(char dec[100])
    {
        char caesar[] = "9(ku@AW1[Lmvgax6q`5Y2Ry?+sF!^HKQiBXCUSe&0M.b%rI'7d)o4~VfZ*{#:}ETt$3J-zpc]lnh8,GwP_ND|jO";     	
        int z, i, j;
        for (i=0; dec[i]!='\0'; i++)
      {
            for (j=0; j<strlen(caesar); j++)
            {
                if(dec[i] == caesar[j])
          {
                    break;
                }
            }
            z = (j-10)%87;
            if(z < 0)
        {
                z = z + strlen(caesar);  
            }
            if(dec[i] == '.' && (strlen(dec)-i)<5)break; 
            else if(dec[i] != '/')
        {
                dec[i] = caesar[z]; 
            }else
        {
                dec[i] = '/'; 
            }
        }
        return dec; 
    }

Fungsi untuk encrypt
Cara kerja fungsi encrypt ialah, dengan mengecek diposisi manakah huruf teks[i] pada huruf caesar (anggap di posisi j), kemudian menggantikan huruf pada teks[i] dengan huruf caesar yang berada pada posisi j+10.

    char* Encrypt(char enc[100])
    {
        char caesar[] = "9(ku@AW1[Lmvgax6q`5Y2Ry?+sF!^HKQiBXCUSe&0M.b%rI'7d)o4~VfZ*{#:}ETt$3J-zpc]lnh8,GwP_ND|jO";
        int i, j;
        for (i=0; enc[i]!='\0'; i++)
      {
            for (j=0; j<strlen(caesar); j++)
            {
                if(enc[i] == caesar[j])
          {
                    break;
                }
            }
            if(enc[i] == '.' && (strlen(enc)-i)<5)break; 
            else if(enc[i] != '/')
        {
                enc[i] = caesar[(j+10)%87]; 
            }else{
                enc[i] = '/';   
            }
        }
        return enc;
    }


Fungsi untuk mengambil nama path dari folder terenkripsi

    char *checkEncrypt(char fpath[100],const char *path)
    {
        int i,j;
        char rev[100],fname[1000]; 
            memset(rev,'\0',100); 
            memset(fname,'\0',1000);
            for (i = 0; i<strlen(path); i++)
            {
                if (path[i]=='/')
                {
                    break;
                }
                rev[i] = path[i]; 
            }
            sprintf(fname,"%s",rev); 
            memset(rev,'\0',100);
            
printf ("i=%d dan rev = %s\n",i,rev);

            j=0; 
            if (i!=strlen(path))
            {
                while (1){
                    rev[j] = path[i]; 
                    i++; 
                    j++; 
                    if(i==strlen(path))
                    {
                        break;
                    }
                }
               
printf ("masuk dan rev = %s\n",rev);

                Encrypt(rev);
                
Decrypt(rev);

                strcat(fname,rev); 
                sprintf(fpath, "%s%s", dirpath, fname); 
                printf ("pathattr1 = %s\n",fpath); 
            }
            else
            {
                sprintf(fpath, "%s%s", dirpath, fname); 
                printf ("pathattr11 = %s\n",fpath); 
            }
        return fpath; 
    }

Fungsi untuk mengambil nama path dari folder terenkripsi dengan '/' pada path[0] 

    char *checkEncryptslash(char fpath[100],const char *path)
    {
        int i,j; 
        char rev[100],fname[1000]; 
            memset(rev,'\0',100); 
            memset(fname,'\0',1000); 
            for (i = 0; i<strlen(path); i++)
            {
                if ((path[i]=='/' && i!=0) || path[i]=='\0')
                {
                    break; 
                }
                rev[i] = path[i]; 
                
mengambil path folder enkripsi
                
            }
            sprintf(fname,"%s",rev); 
            memset(rev,'\0',100); 
            j=0;
            

Jika terdapat foler/file pada folder enkripsi maka nama path diambil           
            if (i!=strlen(path))
            
            
            {
                while (1)
          {
                    rev[j] = path[i]; 
                    i++; 
                    j++; 
                    if(i==strlen(path))
                    {
                        break; 
                    }
                }
                
 Mengenkripsi nama path didalam folder enkripsi
                
                Encrypt(rev);
                
 Menggabungkan fullpath folder enkripsi               
                
                strcat(fname,rev);
                
 Menggabungkan fullpath dengan directory               
                
                sprintf(fpath, "%s%s", dirpath, fname); 
            }
            else
            {
            
Jika tidak terdapat file/folder didalam folder terenkripsi
            
                sprintf(fpath, "%s%s", dirpath, fname); 
            }
        return fpath;
    }
    
Fungsi ini akan mengambil dan mengenkripsi child path dari folder yang terenkripsi. Contoh : encv1_XXX/encrypted/encrypted Kemudian kembali digabung dengan directorypath nya.

Fungsi untuk membuat log

    void createlog(char process[100],char fpath[100])
    {
        char text[200];
        FILE *fp = fopen("/home/nabella/Documents/fs.log","a"); 
        time_t t = time(NULL);
        struct tm tm = *localtime(&t);
        if (strcmp(process,"unlink")==0)
        {
            sprintf(text, "WARNING::%04d%02d%02d-%02d:%02d:%02d::UNLINK::%s\n", tm.tm_year + 1900, tm.tm_mon + 1, tm.tm_mday, tm.tm_hour, tm.tm_min, tm.tm_sec,fpath);  
        }
        else if (strcmp(process,"mkdir")==0)
        {
            sprintf(text, "INFO::%04d%02d%02d-%02d:%02d:%02d::MKDIR::%s\n", tm.tm_year + 1900, tm.tm_mon + 1, tm.tm_mday, tm.tm_hour, tm.tm_min, tm.tm_sec,fpath); 
        }
        else if (strcmp(process,"rmdir")==0)
        {
            sprintf(text, "WARNING::%04d%02d%02d-%02d:%02d:%02d::RMDIR::%s\n", tm.tm_year + 1900, tm.tm_mon + 1, tm.tm_mday, tm.tm_hour, tm.tm_min, tm.tm_sec,fpath); 
        }
        for (int i = 0; text[i] != '\0'; i++) 
      {
                fputc(text[i], fp); 
        }
        fclose (fp);
    }

Fungsi untuk membuat log khusus proses rename

    void createlogrename(char from[100], char to[100])
    {
        FILE *fp = fopen("/home/nabella/Documents/fs.log","a"); 
        time_t t = time(NULL); 
        struct tm tm = *localtime(&t); 
        char text[200]; 
        sprintf(text, "INFO::%04d%02d%02d-%02d:%02d:%02d::RENAME::%s::%s\n", tm.tm_year + 1900, tm.tm_mon + 1, tm.tm_mday, tm.tm_hour, tm.tm_min, tm.tm_sec,from,to);  
        for (int i = 0; text[i] != '\0'; i++) 
      {
                fputc(text[i], fp);  
        }
        fclose(fp);
    }

    static int xmp_getattr(const char *path, struct stat *stbuf)
    {
        int res; 
        char fpath[1000];
        if (strcmp(path, ".")!=0 && strcmp(path, "..")==0)
        {
            sprintf(fpath,"%s%s",dirpath,path); 
        }
        else if (path[0]!='/' && strncmp(path,"encv1_",6)==0)
      {
            checkEncrypt(fpath,path); 
        }
        else if (path[0]=='/' && strncmp(path,"/encv1_",7)==0)
      {
            checkEncryptslash(fpath,path); 
        }
        else
      {
            sprintf(fpath, "%s%s", dirpath, path);  
        }
        res = lstat(fpath, stbuf);  
        if (res == -1)
            return -errno;  
        return 0; 
    }

	static int xmp_readdir(const char *path, void *buf, fuse_fill_dir_t filler, off_t offset, struct fuse_file_info *fi)
	{
	    int flag=0; 
	    char fpath[1000];  
	    if(strcmp(path,"/") == 0)
	    {
		path=dirpath; 
		sprintf(fpath,"%s",path); 
	    }
	    else if (path[0]!='/' && strncmp(path,"encv1_",6)==0)
		{
		flag=1; 
		checkEncrypt(fpath,path); 
	    }
	    else if (path[0]=='/' && strncmp(path,"/encv1_",7)==0)
		{
		flag=1;
		checkEncryptslash(fpath,path); 
	    }
	    else
		{
		sprintf(fpath, "%s%s", dirpath, path); 
	    }
	    int res = 0; 
	    DIR *dp; 
	    struct dirent *de; 
	    (void) offset;
	    (void) fi;
	    dp = opendir(fpath);
	    if (dp == NULL)
		return -errno;
	    while ((de = readdir(dp)) != NULL) 
		{
		struct stat st;
		memset(&st, 0, sizeof(st));
		st.st_ino = de->d_ino;
		st.st_mode = de->d_type << 12; 
		if(strcmp(de->d_name, ".")!=0 && strcmp(de->d_name, "..")!=0 && flag==1)
        
Mendecrypt nama file/folder didalam folder terenkripsi sehingga pada fungsi getattr, nama file akan dienkripsi menghasilkan nama file/folder yang asli        
        
		{
            Decrypt(de->d_name);
        }
        res = (filler(buf, de->d_name, &st, 0));
            if(res!=0) break; 
    }
    closedir(dp);
    return 0; 
    }

    static int xmp_mkdir(const char *path,mode_t mode)
    {
        int res;
        char fpath[1000];
        sprintf(fpath,"%s%s",dirpath,path); 
        res = mkdir (fpath,0777);
        if(res == -1)
            return -errno;
        char temp[] = "mkdir"; 
        createlog(temp,fpath);
        return 0;
    }

    static int xmp_rmdir(const char *path)
    {
        int res;
        char fpath[1000]; 
        if (path[0]!='/' && strncmp(path,"encv1_",6)==0)
      {
            checkEncrypt(fpath,path);
        }
        else if (path[0]=='/' && strncmp(path,"/encv1_",7)==0)
      {
            checkEncryptslash(fpath,path); 
        }
        else
      {
            sprintf(fpath, "%s%s", dirpath, path);
        }
        char temp[] = "rmdir";
        createlog(temp,fpath);
        res = rmdir(fpath); 
        if (res == -1)
            return -errno;
        return 0;
    }

    static int xmp_rename(const char *from, const char *to)
    {
        int res;
        char frname[1000], toname[1000], frpath[1000], topath[1000]; 
        memset(frname, '\0', sizeof(frname));
        memset(toname, '\0', sizeof(toname)); 
        strcpy(frname, from);
        strcpy(toname, to);
        sprintf(frpath, "%s%s", dirpath, frname);
        sprintf(topath, "%s%s", dirpath, toname);
        res = rename(frpath, topath); 
        if (res == -1)
            return -errno;
        createlogrename(frname,toname); 
        return 0; 
    }

    static int xmp_read(const char *path, char *buf, size_t size, off_t offset, struct fuse_file_info *fi)
    {
        char fpath[1000];  
        if(strcmp(path,"/") == 0)
        {
            path=dirpath;
            sprintf(fpath,"%s",path); 
        }
        else if (path[0]!='/' && strncmp(path,"encv1_",6)==0)
      {
            checkEncrypt(fpath,path);
        }
        else if (path[0]=='/' && strncmp(path,"/encv1_",7)==0)
      {
            checkEncryptslash(fpath,path); 
        }
        else{
            sprintf(fpath, "%s%s", dirpath, path); 
        }
        int res = 0;
        int fd = 0 ; 
        (void) fi; 
        fd = open(fpath, O_RDONLY);
        if (fd == -1)
            return -errno; 
        res = pread(fd, buf, size, offset); 
        if (res == -1)
            res = -errno;
        close(fd); 
        return res; 
    }

    static int xmp_unlink(const char *path)
    {
        int res; 
        char fpath[100]; 
        if (path[0]!='/' && strncmp(path,"encv1_",6)==0)
      {
            checkEncrypt(fpath,path); 
        }
        else if (path[0]=='/' && strncmp(path,"/encv1_",7)==0)
      {
            checkEncryptslash(fpath,path);  
        }
        else{
            sprintf(fpath, "%s%s", dirpath, path); 
        }
        char temp[] = "unlink"; 
        createlog(temp,fpath); 
        res = unlink(path); 
        if (res == -1)
            return -errno; 
        return 0; 
    }

    static struct fuse_operations xmp_oper = 
    {
    .getattr    = xmp_getattr,
    .unlink     = xmp_unlink,
    .readdir    = xmp_readdir,
    .rmdir      = xmp_rmdir,
    .mkdir      = xmp_mkdir,
    .read       = xmp_read,
    .rename     = xmp_rename,
    }; 

    int main(int argc, char *argv[])
    {
        umask(0); 
        return fuse_main(argc, argv, &xmp_oper, NULL);  
    }





