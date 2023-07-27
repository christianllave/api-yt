# Rank, Metrics, and Connections from Youtube Data API
## About the project
Youtube has been a prominent player as a platform for video-based creators to build a following. Some creators are motivated by monetization, while others are in it for passion and sharing their topic of interest. Ranking high in Youtube searches and getting recommended by other prominent entities in the content sphere are some of the ways creators can get better traction on their videos; however, information on how the algorithms work is largely hidden within Youtube’s walled gardens. There is plenty of content that talks about how Youtube works from the creator’s perspective, but not a lot of resources provide quantitative explorations on this topic.

This project aims to explore how Youtube’s recommendation and ranking mechanisms work using the Youtube Data API, which allows Google Cloud users to access the data that people would typically see on the Youtube platform, such as videos, search results, channels, and other related features, along with their metadata and metrics. There is much potential to explore multiple dimensions of what makes a video rank high or get connected to other videos; however, this project focuses on the core features, their more quantitative aspects, and the overall process of procuring the data.

The expected output is a set of database files for videos, channels, and search results, with corresponding metadata and metrics for those assets. This allows for basic visualizations of the metrics and connections between related entities.

## Expected Output
The structure of the output allows for flexibility in preprocessing, transforming, and plotting from a range of graphical representations.

Figure 1: Diagram of output data and dependencies between them
![COSC480 Project Report - Llave](https://github.com/christianllave/api-yt/assets/70957302/166b18cb-e54a-42b9-8d8e-cae63fb8c852)

As seen in Figure 1, there are four tables produced as database files:

• **search_results** contains the rank, video_id, and channel_id from the search on the core topic. This is set up with JOIN operations in mind.

• **related_searches** contains the video_id from search_results as the base, and the video_id in this table is the video related to the base. There are multiple related videos for one base. The rank column indicates which position in the search result the video_id was. The assumption is that if they appear first, the rank is higher and is likely more relevant. The ref_count is the number of times the video_id is referenced for that base, or in a way, the strength of the relationship between the videos.

• **video_data** contains all unique video_id values from both search_results and related_searches, as this table is expected to be used for JOIN operations to provide data that search queries lack. This contains relevant metadata and metrics for each video. 

• **channel_data** contains all unique channel_id values from video_data, along with their respective channel metadata. It is expected that this table will be used when making connections between channels when their videos are referenced as related videos.

From the graphs in Figure 2, it can be seen that views and engagement are not the only factors to rank high on Youtube search results. The plots are of different instances of querying the same core topic. It may be good to examine other metadata collected as part of the video data, such as recency, relations, tags, and the nature of the channel.

Figure 2
![Bar Graph](https://github.com/christianllave/api-yt/assets/70957302/d907995c-0e19-4f27-8660-0d8b29170840)

Figure 3 displays a network graph of videos wherein the related videos are clustered around the base video, with some related videos seeming to connect with other bases in between the clusters. The image below plots the first 120 relationships, and has three clusters, with the labels removed for image clarity. 

Figure 3

![Sample network graph v2](https://github.com/christianllave/api-yt/assets/70957302/0a4d8d26-8ba4-4a88-b865-9048af0547e7)


## Development Process
The key objective for this project is to establish a functional set of features while considering limitations set by API access. While the potential to scale data within this project is limited, the idea of future scalability was something that was kept in consideration. In other words, the programs are set up in a way that can be built up by users with higher levels of API access, more intricate data processing, or more interactive graphical needs.

Before the development and coding itself, accessing the API was the first line item. This was done by creating an API Key on the Google Cloud console. The account is free, which comes with its limitations, primarily the request quotas. 

Figure 4 lays out the main steps of the development process. Each file represents a step in accessing the API, storing, and transforming to a database structure until tables are available for visualization. Each file is broken up into smaller functions performing specific tasks to be assembled on the main() function. For files 1-7, the processes are to access API and store results locally. The processes for files 8 and 9 are to access local files, pre-process, and visualize.

Figure 4: Diagram of process flow for the project
![COSC480 Project Report - Llave (1)](https://github.com/christianllave/api-yt/assets/70957302/9207bfa0-6840-423d-8fdb-31acfe055cd2)

### Judgment calls and fine-tuning during development
Files accessing the API (Files 1, 3, 4, 6) have a function designed to access the API need to open a URL endpoint. Support functions are made to create the SSLContext, prompt the search query, declare URL parameters, build the URL, and then open the URL. Additional preprocessing functions like reduce_list in file 3 were created to reduce the list of videos for search requests. Some support functions were designed to be reused, such as create_video_list in file 4, which gets reused for different SQL queries as arguments. 

One mechanism placed on the JSON-to-text storage process is the deletion of text files from past runs. Otherwise, new queries would stack on top of previous search results, and the data would no longer be representative of the study of the core query. This is for this process to be focused on one core query, but that process can be removed in future research that focuses on more core queries. One may argue using a write instead of append when opening the file; however, that would have taken away from experimenting on scalability. 

When storing pulled data from text or JSON into SQLITE, there is an option for the SQL command to either update or insert, which is to enable scaling and updating in future developments. For numerical metrics such as view_count, like_count, and comment_count, they are converted to 0 if they are not present. This is to enable processing.

By combining rank and video_id from search_results with metrics from video_data, a graph of view count and engagement rate for each video arranged by rank can be produced. This was done by joining them using a SQL query and converted to a Pandas dataframe. While not a comprehensive metric of engagement, engagement_rate was calculated by adding like_count and comment_count and then dividing the sum by view_count. This metric was appended to the dataframe, which was then reduced to contain only details relevant to the plot. The reduced dataframe is then plotted on Matplotlib.

To examine connections between videos and related videos, data from related_searches is transformed into a Pandas dataframe, and used to define edges and nodes on a network graph using Networkx. 

