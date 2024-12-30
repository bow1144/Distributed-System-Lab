# 实验一、电影评分数据分析

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

# 实验二、会员群体收入分析

## 一、数据预处理
<img width="355" alt="{FCBCDEC4-1105-4584-B561-83FC53DC14DD}" src="https://github.com/user-attachments/assets/c68afffd-2b03-4903-a302-7fe92bd01636" />

<img width="349" alt="{9439F3E4-928A-4009-B0DC-81EA81DCC575}" src="https://github.com/user-attachments/assets/b7630d93-a884-43a7-9cfa-82bcae914d22" />

<img width="354" alt="{1577E2A1-EBC0-491A-BED1-5DF2DACD81B8}" src="https://github.com/user-attachments/assets/91a8f385-db3d-4503-95cc-e4d82ae27434" />

<img width="369" alt="{99279D5F-E359-4535-9121-576568D45B13}" src="https://github.com/user-attachments/assets/6c712780-5a5c-4cc0-ada6-da2420ff89cf" />

## 二、基本信息可视化
<img width="341" alt="{E3105A5F-B1F2-46A9-BDA5-628CDA38CA81}" src="https://github.com/user-attachments/assets/b370e59e-904f-4f72-8857-95610cac80c3" />

<img width="345" alt="{8679DBC7-BB7B-4F05-9F8A-975C5077EF08}" src="https://github.com/user-attachments/assets/7ea892fc-9cbf-4165-9777-09fceae0ae9b" />

s<img width="348" alt="{E461AB61-978C-4EF4-9759-A8882032A8DA}" src="https://github.com/user-attachments/assets/971463f4-7a4a-4158-bb8b-0b76def6b3fa" />

<img width="350" alt="{9E71FA08-A8AC-4CE5-86DB-36798EC5F3B8}" src="https://github.com/user-attachments/assets/2f7283b6-c656-4474-9d95-5d06f88ceb48" />

<img width="350" alt="{A231431A-D2C4-4512-9F24-496B9B38C45B}" src="https://github.com/user-attachments/assets/f9708cfd-62e9-4e67-829a-d8e13de50ebb" />

## 三、顾客收入解析

<img width="349" alt="{AD0B0B92-6036-4DC3-A783-0EF04B5F9386}" src="https://github.com/user-attachments/assets/b47b25c2-50c8-48f6-bb28-e12828fae371" />

<img width="331" alt="{35DC4D77-905B-4668-A39D-A7A0F6C44B57}" src="https://github.com/user-attachments/assets/f346face-9d48-4029-9c54-bf166036bdc8" />

<img width="347" alt="{1A0E18D3-39AA-4E25-985D-248E4629BFA7}" src="https://github.com/user-attachments/assets/6145617b-d063-44ef-bc74-7301f6fd19e1" />

<img width="352" alt="{E4E755A8-3D29-44EB-AB2C-5AA0723DA24C}" src="https://github.com/user-attachments/assets/6fa2f340-5797-42b1-bf0e-a438bbee5852" />

<img width="345" alt="{00CA2383-28C0-4DFA-826F-483FEB59EE5E}" src="https://github.com/user-attachments/assets/1df4cba8-e77a-4f0d-b2a2-0d2981ef47dc" />

## 四、K-means聚类分析
```
library(purrr)
set.seed(123)
# function to calculate total intra-cluster sum of square 
iss <- function(k) {
  kmeans(customer_data[,3:5],k,iter.max=100,nstart=100,algorithm="Lloyd" )$tot.withinss
}

k.values <- 1:10


iss_values <- map_dbl(k.values, iss)

plot(k.values, iss_values,
    type="b", pch = 19, frame = FALSE, 
    xlab="Number of clusters K",
    ylab="Total intra-clusters sum of squares")
```

<img width="283" alt="{FDF18C65-D9E5-4C4A-89A5-3EBCF8874B83}" src="https://github.com/user-attachments/assets/54e0d19b-0c4f-40a8-937e-505980d48943" />

<img width="346" alt="{D8DC622E-C0F7-40CA-8291-F8FFEEB2281F}" src="https://github.com/user-attachments/assets/1c4ef28f-292e-44fd-aa1a-e13a8b546b71" />

<img width="409" alt="{7694128C-CCC3-4C6F-81E9-7DEA7B42B043}" src="https://github.com/user-attachments/assets/18c9d170-330d-4afe-8366-15bd5013cc07" />

<img width="347" alt="{89FE1C28-6A7C-4363-9BB2-EBD4B2B98409}" src="https://github.com/user-attachments/assets/2bdad71a-e0ac-460b-b569-88c6bd3f6874" />

<img width="354" alt="{9A5FD2B8-441D-41D9-839E-6DDC890AF1A3}" src="https://github.com/user-attachments/assets/2d32406d-6f53-489c-9a0e-13f40d255c1c" />

<img width="359" alt="{7B19E5E1-C1D1-4670-9319-65250A149030}" src="https://github.com/user-attachments/assets/81b9cb94-3b93-4f6c-94c4-e71fd895cd3b" />

<img width="347" alt="{35DF180A-F1FB-4D9A-ABA9-AC3843601937}" src="https://github.com/user-attachments/assets/4cf4dc08-1237-42b4-a106-1f8086e1a30d" />
