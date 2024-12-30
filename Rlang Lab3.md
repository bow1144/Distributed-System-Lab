# 实验一、会员群体画像

## 一、下载数据集
```
setwd("/kaggle/input/movies-dataset")                       #Author DataFlair
movie_data <- read.csv("movies.csv",stringsAsFactors=FALSE)
rating_data <- read.csv("ratings.csv")
str(movie_data)
```

<img width="462" alt="{B1BB053C-D80A-43E8-8223-500F174B1375}" src="https://github.com/user-attachments/assets/0ec3b61f-0e55-41fa-980c-b0bdec5a7416" />

## 二、数据总览
<img width="528" alt="{7B3319FA-DBEB-4BF5-9146-A9A3948B24FA}" src="https://github.com/user-attachments/assets/ecf25d9f-80e8-4187-bb33-c05d0b50df9b" />

<img width="503" alt="{6975FA17-9F40-4CEF-A42A-8B0F6C06A3FF}" src="https://github.com/user-attachments/assets/303b2411-d03f-48e6-905e-6c172e0b7527" />

<img width="509" alt="{A2009463-573F-4F13-B0FD-07CDD4FC205B}" src="https://github.com/user-attachments/assets/e8839d3e-6158-43c9-bedc-14d399e5aee7" />

<img width="492" alt="{DE2B0CF6-C413-4ABC-BFFA-F27CD7FB7F88}" src="https://github.com/user-attachments/assets/36de2228-a202-4117-b1a4-b95489be8fb5" />

## 三、数据预处理
- 从上表中，我们观察到 userId 列和 movieId 列由整数组成。此外，我们需要将 movie_data DataFrame 中存在的流派转换为用户更可用的格式。为此，我们将首先创建一个 one-hot 编码，以创建一个包含每部电影的相应流派的矩阵。

```
movie_genre <- as.data.frame(movie_data$genres, stringsAsFactors=FALSE)
library(data.table)
movie_genre2 <- as.data.frame(tstrsplit(movie_genre[,1], '[|]', 
                                   type.convert=TRUE), 
                         stringsAsFactors=FALSE) #DataFlair
colnames(movie_genre2) <- c(1:10)
list_genre <- c("Action", "Adventure", "Animation", "Children", 
                "Comedy", "Crime","Documentary", "Drama", "Fantasy",
                "Film-Noir", "Horror", "Musical", "Mystery","Romance",
                "Sci-Fi", "Thriller", "War", "Western")
genre_mat1 <- matrix(0,10330,18)
genre_mat1[1,] <- list_genre
colnames(genre_mat1) <- list_genre
for (index in 1:nrow(movie_genre2)) {
  for (col in 1:ncol(movie_genre2)) {
    gen_col = which(genre_mat1[1,] == movie_genre2[index,col]) #Author DataFlair
    genre_mat1[index+1,gen_col] <- 1
  }
}
genre_mat2 <- as.data.frame(genre_mat1[-1,], stringsAsFactors=FALSE) #remove first row, which was the genre list
for (col in 1:ncol(genre_mat2)) {
  genre_mat2[,col] <- as.integer(genre_mat2[,col]) #convert from characters to integers
} 
str(genre_mat2)
```

<img width="392" alt="{85E7C234-B102-476C-90C7-A8BE40C8CA26}" src="https://github.com/user-attachments/assets/c72e6d46-2d01-4090-97b2-575b9173aa52" />

<img width="504" alt="{A538DFCF-9499-490E-800E-E244629DAD15}" src="https://github.com/user-attachments/assets/d337ff3a-de67-4e2f-944f-f7609898e1c5" />

<img width="520" alt="{EDF795F7-2835-4A54-9D4A-FF9B7FED0A1D}" src="https://github.com/user-attachments/assets/1974628e-a4c3-4509-af07-a81d41d7594c" />

<img width="533" alt="{1CBC3830-6624-45CA-80E3-99E84B2C0BEF}" src="https://github.com/user-attachments/assets/56636199-fb1d-4f24-b638-3e17096058ab" />

<img width="508" alt="{06D3A213-526F-4F47-B9B3-7164783DA09A}" src="https://github.com/user-attachments/assets/66bf9b8c-9f39-4d76-9cda-b1bc8a4d3365" />

<img width="480" alt="{129C1C4E-3CC7-442E-AEFA-022140FB3B2A}" src="https://github.com/user-attachments/assets/53651c93-e185-4b48-92a8-56bba3dfc5a8" />

## 四、挖掘相似数据

<img width="500" alt="{C829E037-B6E8-4B46-84BF-7E0FEF6DE682}" src="https://github.com/user-attachments/assets/a95e6dda-c170-49b4-9403-7d9bdbb140cd" />

<img width="506" alt="{B4E28D83-BA7D-472A-ACF9-EE644B900746}" src="https://github.com/user-attachments/assets/99062ba8-0708-432f-983a-faed1c4e2083" />

<img width="513" alt="{B8C34FA4-A7CE-4A1F-89D3-DC9240D1C8E3}" src="https://github.com/user-attachments/assets/5fcabd8d-daf3-4902-9233-5032841a468a" />

<img width="520" alt="{1E6F5442-1ABA-481E-89E9-BD227F3AE971}" src="https://github.com/user-attachments/assets/3edb3dee-c160-4c04-ba29-b99bf1d9bf87" />

<img width="516" alt="{FF026654-B56F-432B-B9D8-884443E532B7}" src="https://github.com/user-attachments/assets/1eeb867a-88fc-4072-acc1-d20bec6b1c67" />

## 五、观看最多电影可视化
<img width="533" alt="{FB4A46C0-21C4-4BBB-A6FE-7F202B44CB73}" src="https://github.com/user-attachments/assets/4d6d10d4-58bb-498b-8f03-83199d43b91e" />

<img width="536" alt="{22246DC2-D5CB-44D5-8BB0-09AB658442B2}" src="https://github.com/user-attachments/assets/ab02e187-83e1-4e01-9cb4-bc5fca8beb67" />

## 六、电影评分热力图

<img width="515" alt="{21A2AA28-4CC2-4913-964A-8B8ECE3340A9}" src="https://github.com/user-attachments/assets/ec9f0539-3b7f-4f35-931e-82647e6d03e9" />

<img width="517" alt="{5F53F227-976E-4F19-9029-2BAB15ED263B}" src="https://github.com/user-attachments/assets/d721aa64-a3ff-499a-921d-618b79e94b36" />

<img width="510" alt="{E1794C59-1477-47AA-BFFE-C674A671ADF2}" src="https://github.com/user-attachments/assets/10fd6a30-89c3-409b-aa46-9d1b2a8cb0ec" />

<img width="514" alt="{A794D2AB-DBB9-4EF0-8B01-86E7603A629A}" src="https://github.com/user-attachments/assets/d6699954-9c5d-4d4b-a81b-a543b34f444d" />

## 七、数据规范化
<img width="531" alt="{4F4BBBE8-A476-4CC5-978A-9B9EA4145C00}" src="https://github.com/user-attachments/assets/4340f02d-71b0-479d-b959-604169259035" />

<img width="529" alt="{8EE66E3E-65C2-4878-91EF-A2FE170762F5}" src="https://github.com/user-attachments/assets/dd425f1b-3721-4f10-ad29-e2bb62820feb" />

## 八、协同过滤系统
<img width="516" alt="{3E4033CF-BE2E-44F8-8A24-5F26AD287C06}" src="https://github.com/user-attachments/assets/56083386-dbd4-42c0-bd91-109a241018b9" />

<img width="526" alt="{92D752C0-3979-47F8-8A26-3658AEDD1CDF}" src="https://github.com/user-attachments/assets/0f6449a9-2b6c-470b-8918-94b1d8cbca10" />

<img width="517" alt="{2498A604-8F18-4584-9796-C10CD65222DF}" src="https://github.com/user-attachments/assets/6293f4d9-a24e-4a26-b323-5f27cbdd43d9" />

<img width="362" alt="{D19ECF51-836D-4A22-9BCF-CC12962918D9}" src="https://github.com/user-attachments/assets/0969b03c-afd5-4ad0-a1cf-1b374dc51ca9" />

<img width="513" alt="{E0A2D4F9-522C-4CFB-A36E-322857405694}" src="https://github.com/user-attachments/assets/dce4f6c5-6884-494f-a5e0-75303deb5dbe" />

<img width="547" alt="{941EB8ED-D583-4DEB-AFF3-7A91AEA0D4B1}" src="https://github.com/user-attachments/assets/936d5ef7-bc29-48bf-b50d-2fb9a9366ea1" />

<img width="518" alt="{A715C4C7-7733-4288-A282-38739C9B9F6C}" src="https://github.com/user-attachments/assets/a8c9d0ab-a93c-4d1f-af11-139f4f4f6658" />
