<template>
    <div class="view">
        <div class="check">
            <div class="movieName">
                <p style="color: rgb(255, 255, 255);">電影名稱：</p>
                <input type="text" v-model="this.movieName" style="width: 82%;">
            </div>
            <!-- <div class="onDate">
                <p>上映日期:</p>
                <input type="date" name="" id="" v-model="this.onDate">
            </div> -->
            <div class="click">
                <button class="buttonZ" type="submit" @click="getPlayMovie()">正在熱映</button>
                <button class="buttonZ" type="submit" @click="getMovieName()">搜尋</button>
            </div>
        </div>
        <div style="display: flex;width: 90vw;">
            <i class="fa-solid fa-rotate-left icon" @click="gobackSearch()"></i>
            <p class="textTC">點此返回</p>
        </div>
        <table class="table1">
            <thead>
                <tr class="index">
                    <th>電影海報</th>
                    <th style="width: 5vw;">電影名稱</th>
                    <!-- <th>電影類型</th> -->
                    <th style="width: 46vw;">電影描述</th>
                    <th style="width: 10vw;">上映日期</th>
                    <th style="width: 7vw;">新增</th>

                </tr>
                <tr v-for="(movie, index) in displayedMovies " :key="index">

                    <td style="width: 200px; background-color: #6f81a2;"><img
                            :src="'https://image.tmdb.org/t/p/w342' + movie.poster_path" alt="" @click="gotoSeat(movie)"
                            style="width: 200px; ">
                    </td>
                    <td style="background-color: #6f81a2;">{{ movie.title }}</td>
                    <!-- <td>{{ getType(movie) }}</td> -->
                    <td style=" width: 35vw;background-color: #6f81a2;">{{ truncateOverview(movie.overview === "" ? "尚未有簡介"
                        :
                        movie.overview) }}</td>
                    <td style="margin: 0 2% 0 2%;background-color: #6f81a2;">{{ movie.release_date }}</td>
                    <td style="background-color: #6f81a2;"><button class="create" type="button"
                            @click="gotoSeat(movie)">建立場次</button></td>
                </tr>
            </thead>
        </table>
        <div class="pagination">
            <button @click="prevPage" :disabled="currentPage === 1">上一頁</button>
            <button v-for="pageNumber in pageNumbers" :key="pageNumber" @click="goToPage(pageNumber)"
                :class="{ 'active-page': pageNumber === currentPage }">{{ pageNumber }}</button>
            <button @click="nextPage" :disabled="currentPage === totalPages">下一頁</button>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
export default {
    data() {
        return {
            //電影相關資料
            objPlayMovies: [],
            movieName: "",
            onDate: "",
            //類型
            objtype: [],
            //上下頁以及一頁有幾個的變數
            currentPage: 1,
            pageSize: 10,
            pageNumber: "",
            //語言標籤
            languageTarget: "zh-TW",
            target: "",
        }
    },
    methods: {
        //限制簡介的數字
        truncateOverview(overview) {
            const maxLength = 200; // 设置最大字数
            if (overview.length > maxLength) {
                return overview.substring(0, maxLength) + "...";
            } else {
                return overview;
            }
        },
        //滑到最上面
        scrollToTop() {
            window.scrollTo(0, 0);
        },
        //到指定頁數
        goToPage(pageNumber) {
            this.currentPage = pageNumber;
            this.scrollToTop()
        },
        //下一頁
        nextPage() {
            if (this.currentPage < Math.ceil(this.objPlayMovies.length / this.pageSize)) {
                this.currentPage++;
            }
            this.scrollToTop()
        },
        //上一頁
        prevPage() {
            if (this.currentPage > 1) {
                this.currentPage--;
            }
            this.scrollToTop()
        },
        //回到搜尋頁面
        gobackSearch() {
            this.$router.push('/backSearch')
        },
        //到添加資訊頁面
        gotoSeat(movie) {
            console.log(movie)
            this.$router.push({
                name: 'backAdd',
                query: {
                    movieId: movie.id,
                    movieTitle: movie.title,
                }
            });
        },
        async getPlayMovie() { //上映中
            const options = {
                method: "GET",
                headers: {
                    accept: "application/json",
                    Authorization:
                        "Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiIxZTBiNGVhYWYyMjVhZTdmYzFhNjdjYzk0ODk5Mjk5OSIsInN1YiI6IjY1N2ZjYzAzMGU2NGFmMDgxZWE4Mjc3YSIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.3d6GcXTBf2kwGx9GzG7O4_8eCoHAjGxXNr9vV1lVXww",
                },
            };
            let page = 1;
            let count = 30; //要抓的電影數
            let playingMovies = [];

            try {
                const nowDate = new Date();
                const twoMonth = new Date();
                twoMonth.setMonth(nowDate.getMonth() - 2);
                while (playingMovies.length < count) {
                    const api = `https://api.themoviedb.org/3/movie/now_playing?language=zh-TW&page=${page}`;
                    const response = await fetch(api, options);

                    if (!response.ok) {
                        throw new Error("Network response was not ok");
                    }
                    const data = await response.json();
                    const moviesOnPage = data.results.filter((movie) => {
                        const releaseDate = new Date(movie.release_date);
                        // 檢查發佈日期是否在指定範圍內
                        if (!(releaseDate >= twoMonth && releaseDate <= nowDate)) {
                            return false;
                        }
                        // 檢查poster_path是否存在
                        if (!movie.poster_path) {
                            return false;
                        }
                        return true;
                    });
                    // 移除已存在的電影，避免重複
                    for (const movie of moviesOnPage) {
                        if (!playingMovies.some((existingMovie) => existingMovie.title === movie.title)) {
                            playingMovies.push(movie);
                        }
                    }
                    if (page < data.total_pages) {
                        page++;
                    } else {
                        break;
                    }
                }
                // 過濾掉沒有 poster_path 的電影
                const playMovies = playingMovies.filter((movie) => movie.poster_path).slice(0, count).sort((a, b) => new Date(a.release_date) - new Date(b.release_date));
                this.objPlayMovies = playMovies;
                console.log('上映中 PlayMovies:', this.objPlayMovies);
            } catch (error) {
                console.error(error);
            }
        },
        async getMovieName() { //上映中
            const options = {
                method: "GET",
                headers: {
                    accept: "application/json",
                    Authorization: 'Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJkNTFmNDFjYjUxYWI2NmIzMjJkMGM1OGZkMDY1Y2I1YSIsInN1YiI6IjY1NThmNzFmMDgxNmM3MDBhYmJlNWQ3MCIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.RtMbqdUQUCfdqaLD5SoZ18e4PlSq9Ap4ShtGhmUMm10'
                },
            };

            let page = 1;
            let count = 30; //要抓的電影數
            let playingMovies = [];

            try {
                while (playingMovies.length < count) {
                    const api = `https://api.themoviedb.org/3/search/movie?query=${this.movieName}&include_adult=false&language=${this.languageTarget}&page=${page}`;
                    const response = await fetch(api, options);

                    if (!response.ok) {
                        throw new Error("Network response was not ok");
                    }
                    const data = await response.json();
                    const moviesOnPage = data.results.filter((movie) => {
                        // 檢查poster_path是否存在
                        if (!movie.poster_path) {
                            return false;
                        }
                        return true;
                    });
                    // 移除已存在的電影，避免重複
                    for (const movie of moviesOnPage) {
                        if (!playingMovies.some((existingMovie) => existingMovie.title === movie.title)) {
                            playingMovies.push(movie);
                        }
                    }
                    if (page < data.total_pages) {
                        page++;
                    } else {
                        break;
                    }
                }
                // 過濾掉沒有 poster_path 的電影
                const playMovies = playingMovies.filter((movie) => movie.poster_path).slice(0, count).sort((a, b) => new Date(a.release_date) - new Date(b.release_date));
                this.objPlayMovies = playMovies;
                console.log('上映中 PlayMovies:', this.objPlayMovies);
            } catch (error) {
                console.error(error);
            }
        },

        //尚未解決
        // getMovieType() { //電影類型
        //     const options = {
        //         method: "GET",
        //         headers: {
        //             accept: "application/json",
        //             Authorization:
        //                 "Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiIxZTBiNGVhYWYyMjVhZTdmYzFhNjdjYzk0ODk5Mjk5OSIsInN1YiI6IjY1N2ZjYzAzMGU2NGFmMDgxZWE4Mjc3YSIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.3d6GcXTBf2kwGx9GzG7O4_8eCoHAjGxXNr9vV1lVXww",
        //         },
        //     };
        //     fetch(
        //         "https://api.themoviedb.org/3/genre/movie/list?language=en",
        //         options
        //     )
        //         .then((response) => response.json())
        //         .then((response) => {
        //             (this.type = response.genres),
        //                 console.log(this.type);
        //             this.objtype = this.type;
        //         })
        //         .catch((err) => console.error(err));
        // },
        // getType(movie) {
        //     console.log(movie.genre_ids[1]);
        //     console.log();
        //     for (let i = 0; i < movie.genre_ids.length; i++) {
        //         for (let j = 0; j < this.objtype.length; j++)
        //             if (parseInt(movie.genre_ids[i]) === this.objtype[j].id) {
        //                 types.push(this.objPlayMovies[j].name);
        //             }
        //         // 在這裡將電影類型映射到 movieTypeMap 中
        //         this.$set(this.movieTypeMap, movie.id, types);

        //         // 返回電影類型
        //         return types;
        //     }
        // },

    },
    mounted() {
        this.getPlayMovie()
        // this.getMovieType()
    },
    computed: {
        displayedMovies() {
            const startIndex = (this.currentPage - 1) * this.pageSize;
            const endIndex = startIndex + this.pageSize;
            return this.objPlayMovies.slice(startIndex, endIndex);
        },
        totalPages() {
            return Math.ceil(this.objPlayMovies.length / this.pageSize);
        },
        pageNumbers() {
            return Array.from({ length: this.totalPages }, (_, index) => index + 1);
        },
    },
}
</script>


<style scoped lang="scss">
.view {

    display: flex;
    flex-direction: column;
    align-items: center;
    background-color: rgb(174, 177, 192);
    // background-color: rgb(255, 255, 255);

    .check {
        display: flex;
        width: 90vw;
        height: 12vh;
        // border: 1px solid black;
        border-radius: 10px;
        margin-top: 10px;
        background-color: #525f75;

        .movieName {
            width: 60vw;
            display: flex;
            justify-content: start;
            align-items: center;
            font-size: 16pt;
            margin-left: 20px;
            color: rgb(0, 0, 0);

            input {
                width: 80%;
                height: 5vh;
                margin-left: 20px;
            }

            p {
                margin: 0;
            }
        }

        .onDate {
            width: 50vw;
            display: flex;
            justify-content: start;
            align-items: center;
            font-size: 16pt;
            margin-left: 50px;

            input {
                height: 5vh;
                width: 50%;
                margin-left: 3vw;
                font-size: 16pt;
            }
        }

        .click {
            height: 10vh;
            width: 30vw;

            .search {
                width: 30%;
                height: 5vh;
                margin-top: 3.5vh;
                color: rgb(0, 0, 0);
                background-color: salmon;
            }

            .search1 {
                width: 30%;
                height: 5vh;
                color: rgb(0, 0, 0);
                background-color: salmon;
                margin-right: 35px;
            }

            p {
                margin: 0;
            }
        }
    }


    .icon {
        display: flex;
        width: 60px;
        height: 9vh;
        font-size: 40pt;
        // line-height: 2em;
        padding-top: 10px;
    }

    .textTC {
        font-family: 'jf-openhuninn-2.0';
        font-size: 1.5em;
        margin-top: 20px;
        color: rgb(0, 0, 0);
        // margin-bottom: 50px;
    }


    .table1 {
        width: 90vw;
        height: auto;

        .index {
            background-color: #525f75;

        }

        img {
            cursor: pointer;
            width: 100%;
            /* 使圖片寬度滿格 */
            object-fit: cover;
            /* 保持圖片比例並填滿td */
        }

        th {
            height: 5vh;
            border: 1px solid black;
            border-radius: 5px;
            color: rgb(237, 235, 235);

        }

        td {
            border: 1px solid black;
            color: rgb(0, 0, 0);
            font-size: 16pt;

        }

        .create {
            border: 0;
            font-size: 1em;
            font-family: 'jf-openhuninn-2.0';
            color: rgb(0, 0, 0);
            margin-top: 2.5%;
            transition: 0.4s;
            line-height: 1em;
            border: none;
            background: none;
            outline: none;
            background-color: none;

            &:hover {
                color: rgb(228, 220, 220);
                transform: scale(1.2, 1.2);
            }
        }
    }

    .pagination {
        width: 90vw;
        justify-content: center;
        align-items: center;
        background-color: #525f75;

        button {
            color: rgb(255, 255, 255);
            font-size: 18pt;
            background-color: #525f75;
            border: 0px;

            &.active-page,
            &:hover {
                color: rgb(0, 0, 0); // 上一页、下一页的颜色
            }
        }

    }
}

.buttonZ {
    width: 12.2vw;
    height: 4.9vh;
    border: none;
    background-color: rgb(176, 182, 213);
    border-radius: 10px;
    font-size: 1.2em;
    font-family: 'jf-openhuninn-2.0';
    transition: 0.4s;
    line-height: 1em;
    margin: 25px 30px 0 0;

    &:hover {
        background-color: gainsboro;
        color: darkslategray;
        transform: scale(1.1, 1.1);
    }
}


.fixword {
    width: 6.2vw;
    height: 3.9vh;
    border: none;
    font-size: 1em;
    font-family: 'jf-openhuninn-2.0';
    color: white;
    margin-top: 2.5%;
    transition: 0.4s;
    line-height: 1em;
    border: none;
    background: none;
    outline: none;

    &:hover {
        color: rgb(255, 255, 255);
        transform: scale(1.2, 1.2);
    }
}</style>