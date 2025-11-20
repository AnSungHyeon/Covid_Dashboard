import streamlit as st
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.font_manager as fm
import os
import matplotlib.font_manager as fm
import matplotlib.pyplot as plt
import platform

# 한글 폰트 자동 설정 함수
def set_korean_font():
    system = platform.system()

    if system == "Darwin":   # macOS
        plt.rcParams['font.family'] = 'AppleGothic'
    elif system == "Windows":
        plt.rcParams['font.family'] = 'Malgun Gothic'
    else:
        # Linux, Streamlit Cloud 등
        plt.rcParams['font.family'] = 'NanumGothic'

    plt.rcParams['axes.unicode_minus'] = False

@st.cache_data
def load_data():
    # 1) CSV 읽기 (N/A를 결측값으로 인식)
    df = pd.read_csv("covid_worldwide.csv", na_values=["N/A"])

    # 2) 컬럼 이름 앞뒤 공백 제거 (예: "Country " → "Country")
    df.columns = df.columns.str.strip()

    # 3) Country 문자열 처리
    if "Country" not in df.columns:
        st.error("CSV 파일에 'Country'라는 컬럼이 없습니다. 현재 컬럼: " + ", ".join(df.columns))
        st.stop()

    df["Country"] = df["Country"].astype(str).str.strip()

    # 4) N/A(결측) 포함된 행 삭제
    df = df.dropna()

    # 5) Country가 빈 문자열인 행 삭제
    df = df[df["Country"] != ""]

    # 6) 숫자형 컬럼 숫자로 변환
    num_cols = [
        "Total Cases",
        "Total Deaths",
        "Total Recovered",
        "Active Cases",
        "Total Test",
        "Population",
    ]

    for col in num_cols:
        if col in df.columns:
            # 쉼표 제거 후 숫자로 변환
            df[col] = (
                df[col]
                .astype(str)
                .str.replace(",", "", regex=False)
                .str.strip()
            )
            df[col] = pd.to_numeric(df[col], errors="coerce")
        else:
            st.error(f"CSV 파일에 '{col}' 컬럼이 없습니다. 현재 컬럼: " + ", ".join(df.columns))
            st.stop()

    # 숫자 변환 후 NaN 생긴 행 또 제거
    df = df.dropna(subset=num_cols)

    # 인덱스 리셋
    df = df.reset_index(drop=True)

    return df

# 타이틀 및 웹페이지 가로로 사용
st.set_page_config(page_title="COVID-19 World Dashboard",page_icon="🌏",layout="wide")
set_korean_font()

# 전세계 데이터 또는 대한민국 데이터 선택
world_data_checkbox = st.sidebar.checkbox("Worldwide Data")

if world_data_checkbox:

    st.sidebar.markdown("### 📌 Show/Hide Options")
    show_preview = st.sidebar.checkbox("데이터 미리보기")
    show_cases_ratio = st.sidebar.checkbox("인구수 대비 감염자 비율(상위20개국)")
    show_death_ratio = st.sidebar.checkbox("감염자 대비 사망자 비율(상위20개국)")
    show_recover_ratio = st.sidebar.checkbox("감염자 대비 회복인원 비율(상위20개국)")
    show_worldmap = st.sidebar.checkbox("전 세계 누적 확진자 지도")

    def main():
        
        st.title("🌍 COVID-19 세계 감염 현황 대시보드")

        df = load_data()

        if show_preview:
        # 디버깅용: 데이터 미리보기
            st.markdown("#### 🔍 데이터 미리보기")
            st.dataframe(df.head())

            st.markdown(f"총 행 개수: **{len(df)}**, 총 국가 수: **{df['Country'].nunique()}**")


        if show_cases_ratio:
            df_case = df[df["Total Cases"] >= 1000].copy()

            df_case["Cases per 100k"] = (df_case["Total Cases"] / df_case["Population"]) * 100000

            st.markdown("#### 👥 인구수 대비 누적 감염자 수 (상위 20개국, 인구 10만 명당)")

            top20_case = df_case.sort_values("Cases per 100k", ascending=False).head(20)

            fig_ratio = px.bar(
                top20_case,
                x="Country",
                y="Cases per 100k",
                hover_data=["Total Cases", "Population"],
                labels={"Cases per 100k": "Cases per 100,000 people"},
                title="인구 10만 명당 누적 확진자 수 상위 20개국",
            )

            fig_ratio.update_layout(xaxis_tickangle=-45)

            st.plotly_chart(fig_ratio, use_container_width=True)
        
        if show_death_ratio:
            st.markdown("#### ☠️ 감염자 대비 사망자 비율 (치명률 상위 20개국)")

            # 감염자가 최소 1000명 이상인 데이터를 사용
            df_cfr = df[df["Total Cases"] >= 1000].copy()

        
            df_cfr["CFR (%)"] = (df_cfr["Total Deaths"] / df_cfr["Total Cases"]) * 100

            top20_cfr = df_cfr.sort_values("CFR (%)", ascending=False).head(20)

            fig_cfr = px.bar(
                top20_cfr,
                x="Country",
                y="CFR (%)",
                hover_data=["Total Cases", "Total Deaths"],
                text = "CFR (%)",
                labels={"CFR (%)": "Case Fatality Rate (%)"},
                title="감염자 대비 사망자 비율 상위 20개국 (치명률), Total Cases >= 1000",
            )

            fig_cfr.update_traces(
                texttemplate='%{text:.2f}%',            
                textposition='outside'                    
            )
            max_val = top20_cfr["CFR (%)"].max()
            fig_cfr.update_layout(
                yaxis_range=[0, max_val * 1.15],    
                xaxis_tickangle=-45
            )

            st.plotly_chart(fig_cfr, use_container_width=True)

        if show_recover_ratio:
            st.markdown("### 💉 감염자 대비 회복인원 비율(회복률 하위 20개국)")

            df_recovered = df[df["Total Cases"] >= 1000].copy()

            df_recovered["Recovered (%)"] = (df_recovered["Total Recovered"] / df_recovered["Total Cases"]) *100

            top20_recovery = df_recovered.sort_values("Recovered (%)", ascending=True).head(20)

            fig_cfr = px.bar(
                top20_recovery,
                x="Country",
                y="Recovered (%)",
                hover_data=["Total Cases", "Total Recovered"],
                text="Recovered (%)",                    
                labels={"Recovered (%)": "Recovered Ratio (%)"},
                title="감염자 대비 회복 인원 비율 하위 20개국 (회복률), Total Cases >= 1000",
            )

            fig_cfr.update_traces(
                texttemplate='%{text:.2f}%',            
                textposition='outside'                    
            )
            max_val = top20_recovery["Recovered (%)"].max()
            fig_cfr.update_layout(
                yaxis_range=[0, max_val * 1.15],    
                xaxis_tickangle=-45
            )

            st.plotly_chart(fig_cfr, use_container_width=True)

        if show_worldmap:

            col_map, col_detail = st.columns([2, 1])

            # =======================
            # 왼쪽: 세계 지도
            # =======================
            with col_map:
                st.subheader("🗺 전세계 누적 확진자 지도")

                # Plotly choropleth (나라 이름 기반)
                fig = px.choropleth(
                    df,
                    locations="Country",              # Country 컬럼 사용
                    locationmode="country names",     # 나라 이름 모드
                    color="Total Cases",
                    hover_name="Country",
                    color_continuous_scale="Reds",
                    projection="natural earth",
                    labels={"Total Cases": "Total Cases"},
                )

                fig.update_layout(
                    margin=dict(l=0, r=0, t=0, b=0),
                    coloraxis_colorbar=dict(title="Total Cases")
                )

                st.plotly_chart(fig, use_container_width=True)

            # =======================
            # 오른쪽: 국가 선택 + 지표
            # =======================
            with col_detail:
                st.subheader("📊 국가별 상세 현황")

                countries = sorted(df["Country"].unique().tolist())

                if not countries:
                    st.error("Country 데이터가 비어 있습니다.")
                    st.stop()

                selected_country = st.selectbox("국가를 선택하세요", countries, index=0)

                # 선택된 국가에 해당하는 행 찾기 (안전하게)
                mask = df["Country"] == selected_country
                country_df = df.loc[mask]

                if country_df.empty:
                    st.error(f"'{selected_country}' 국가에 대한 데이터가 없습니다.")
                    st.stop()

                # 첫 번째 행만 사용
                row = country_df.iloc[0]

                st.markdown(f"### {selected_country}")

                c1, c2 = st.columns(2)
                c3, c4 = st.columns(2)

                c1.metric("Total Cases", f"{int(row['Total Cases']):,}")
                c2.metric("Total Deaths", f"{int(row['Total Deaths']):,}")
                c3.metric("Total Recovered", f"{int(row['Total Recovered']):,}")
                c4.metric("Active Cases", f"{int(row['Active Cases']):,}")

                st.markdown("### 🧪 검사 및 인구")
                p1, p2 = st.columns(2)
                p1.metric("Total Test", f"{int(row['Total Test']):,}")
                p2.metric("Population", f"{int(row['Population']):,}")

    if __name__ == "__main__":
        main()

# ===========================================================================================================================
# ===========================================================================================================================
# ===========================================================================================================================
# ===================================================전 세계 데이터 부분 끝 ====================================================
# ===========================================================================================================================
# ===========================================================================================================================
# ===========================================================================================================================

south_korea_data_checkbox = st.sidebar.checkbox("South Korea Data")

if south_korea_data_checkbox:
    st.title("🇰🇷 South Korea COVID-19 Dashboard")

    st.sidebar.markdown("### 📌 Show/Hide Options")
    show_moving_k = st.sidebar.checkbox("3개월 이동평균 · 월별 국내발생 & 해외유입")
    show_cfr_k = st. sidebar.checkbox("누적 확진자 및 사망자")
    show_age_ConfirmedCase = st.sidebar.checkbox("연령대별 누적 확진자 수")
    show_age_dead = st.sidebar.checkbox("연령대별 누적 사망자 수")
    show_region = st.sidebar.checkbox("시도별 코로나 발생수 지도 산점도")
    
    # ================================================================
    # ========================은영님 부분 그래프=========================
    # ================================================================

    # 대한민국 3개월 이동평균 및 월별 국내발생, 해외유입
    if show_moving_k:
        file_name = '일별 국내 & 해외.csv'
        try:
            df = pd.read_csv(file_name, encoding='utf-8-sig')
        except FileNotFoundError:
            st.error("'일별 국내 & 해외.csv' 파일을 찾을 수 없습니다.")
            st.stop()
        except Exception:
            df = pd.read_csv(file_name, encoding='cp949')

        df.columns = df.columns.str.strip()
        df['일자'] = pd.to_datetime(df['일자'])

        cols_to_numeric = ['국내발생(명)', '해외유입(명)']
        for col in cols_to_numeric:
            df[col] = df[col].astype(str).str.replace(",", "", regex=False)
            df[col] = df[col].replace("-", "0")

        df[cols_to_numeric] = df[cols_to_numeric].apply(pd.to_numeric)
        df = df.set_index('일자')
        
        df_monthly_sum = df[cols_to_numeric].resample("M").sum()
        df_smooth = df_monthly_sum.rolling(window=3, min_periods=1).mean()

        st.subheader("📅 3개월 이동평균 · 월별 국내발생 & 해외유입")
        st.write("**그래프 설명:** 아래 그래프는 국내 발생 및 해외 유입 확진자 수를 월별로 합산하고, 3개월 이동평균을 적용한 추세선입니다.")
        st.write("---")

        # 그래프 그리기
        
        df_plotly = df_smooth.reset_index()
        
        fig = px.line(
            df_plotly,
            x='일자',
            y=['국내발생(명)', '해외유입(명)'], 
            title="월별 국내 발생 및 해외 유입 확진자 수 (3개월 이동평균)",
            labels={'일자': '월(Month)', 'value': '월별 확진자 수(명)', 'variable': '구분'}
        )

        fig.update_yaxes(tickformat=",,.0f") 
        
        fig.update_traces(
            hovertemplate="<b>%{x|%Y년 %m월}</b><br>%{data.name}: %{y:,.0f} 명<extra></extra>"
        )
                
        st.plotly_chart(fig, use_container_width=True)
    
    # 대한민국 감염자 및 사망자
    if show_cfr_k:
        st.subheader("📌 시도별 누적 확진자 및 사망자")

        # 그래프 설명
        st.write("**그래프 설명:** 시도별 누적 확진자 수(좌측 축, 10만 명 단위)와 누적 사망자 수(우측 축, 명)를 함께 비교한 이중 축 막대그래프입니다.")
        st.write("**X축:** 시도명 · **왼쪽 Y축:** 누적 확진자(10만 명) · **오른쪽 Y축:** 누적 사망자(명)")
        st.write("---")

        plt.rc("font", family="Malgun Gothic")
        plt.rcParams["axes.unicode_minus"] = False

        # 데이터 전처리
        file_name = "누적.csv"
        try:
            df = pd.read_csv(file_name, encoding="utf-8-sig")
        except FileNotFoundError:
            st.error("'누적.csv' 파일을 찾을 수 없습니다.")
            st.stop()
        except Exception:
            df = pd.read_csv(file_name, encoding="cp437") 

        df.columns = df.columns.str.strip()
        if "계" in df.iloc[0].values:
            df_cleaned = df.drop(index=0).reset_index(drop=True)
        else:
            df_cleaned = df.reset_index(drop=True)
        df_cleaned = df_cleaned.rename(columns={"시도명": "구분"})
        cols_to_numeric = ["누적확진자(명)", "누적사망자(명)"]
        for col in cols_to_numeric:
            df_cleaned[col] = df_cleaned[col].astype(str).str.replace(",", "", regex=False)
            df_cleaned[col] = df_cleaned[col].replace("-", "0")
        df_cleaned[cols_to_numeric] = df_cleaned[cols_to_numeric].apply(pd.to_numeric)
        df_agg = (
            df_cleaned.groupby("구분")[["누적확진자(명)", "누적사망자(명)"]]
            .sum().reset_index()
        )
        df_agg = df_agg[df_agg["구분"] != "검역"].copy()
        df_sorted = df_agg.sort_values(by="누적확진자(명)", ascending=False)
        
        # 그래프 그리기 
        
        x_indices = np.arange(len(df_sorted))
        x_labels = df_sorted['구분']

        fig = make_subplots(specs=[[{"secondary_y": True}]])
        
        fig.add_trace(
            go.Bar(
                x=x_indices - 0.2,
                y=df_sorted['누적확진자(명)'] / 100000, # 10만 단위로 스케일링
                name='누적 확진자 (10만 명)',
                marker_color='royalblue',
                width=0.4,
                customdata=df_sorted[['구분', '누적확진자(명)']], 
                hovertemplate="<b>%{customdata[0]}</b><br>누적 확진자: %{customdata[1]:,d} 명<extra></extra>"
            ),
            secondary_y=False,
        )

        fig.add_trace(
            go.Bar(
                x=x_indices + 0.2,
                y=df_sorted['누적사망자(명)'],
                name='누적 사망자 (명)',
                marker_color='crimson',
                width=0.4,
                customdata=df_sorted[['구분', '누적사망자(명)']],
                hovertemplate="<b>%{customdata[0]}</b><br>누적 사망자: %{customdata[1]:,d} 명<extra></extra>"
            ),
            secondary_y=True,
        )

        # 4. 레이아웃 및 축 설정
        fig.update_layout(
            title_text='시도별 누적 확진자 및 사망자',
            xaxis=dict(
                tickmode='array',
                tickvals=x_indices,
                ticktext=x_labels
            ),
            xaxis_tickangle=0,
            legend_title_text='범례'
        )
        
        # Y축 제목 설정
        fig.update_yaxes(title_text="누적 확진자 (10만 명)", secondary_y=False)
        fig.update_yaxes(title_text="누적 사망자 (명)", secondary_y=True)

        st.plotly_chart(fig, use_container_width=True)


    # ================================================================
    # ========================동희님 부분 그래프=========================
    # ================================================================


    def main():

        df = load_data()

        if show_age_ConfirmedCase:
            # 그래프 설명 추가
            st.markdown("### 👤 연령대별 누적 확진자 수")
            st.write("이 그래프는 국내 코로나19 확진자 수를 연령대별로 집계한 것입니다.")
            st.write("**X축:** 연령대 · **Y축:** 누적 확진자 수(명)")
            st.write("---")

            x = ['0-9세','10-19세','20-29세','30-39세','40-49세','50-59세','60-69세','70-79세','80세이상']
            y = [3270282, 4246977, 5001143, 5077726, 5237546, 4531012, 3898836, 2056083, 1252949]

            fig, ax = plt.subplots(figsize=(10, 6))
            ax.bar(x, y)
            ax.set_xticklabels(x, rotation=45)
            for p in ax.patches:
                ax.text(p.get_x() + (p.get_width()/2) ,   # 가로 위치
                        p.get_y() + p.get_height(),   # 세로 위치
                        f"{p.get_height()}명",     # 값 + 표시방법 소수 둘째자리까지 
                        ha = 'center' )   # 좌우정렬 중간으로
            ax.set_title('연령대별 누적 확진자 수')
            ax.set_xlabel('연령대')
            ax.set_ylabel('누적 확진자 수')
            fig.tight_layout()
            st.pyplot(fig)

        if show_age_dead:
            # 설명 추가
            st.markdown("### ⚰️ 연령대별 누적 사망자 수")
            st.write("이 그래프는 국내 코로나19 사망자 수를 연령대별로 집계한 것입니다.")
            st.write("**X축:** 연령대 · **Y축:** 누적 사망자 수(명)")
            st.write("---")

            x_dead = ['0-9세','10-19세','20-29세','30-39세','40-49세','50-59세','60-69세','70-79세','80세이상']
            y_dead = [38, 24, 73, 160, 473, 1422, 4008, 8062, 21345]

            fig, ax = plt.subplots(figsize=(10, 6))
            ax.bar(x_dead, y_dead)
            ax.set_xticklabels(x_dead, rotation=45)
            for p in ax.patches:
                ax.text(p.get_x() + (p.get_width()/2) ,   # 가로 위치
                        p.get_y() + p.get_height(),   # 세로 위치
                        f"{p.get_height()}명",     # 값 + 표시방법 소수 둘째자리까지 
                        ha = 'center' )   # 좌우정렬 중간으로
            ax.set_title('연령대별 사망자 수')
            ax.set_xlabel('연령대')
            ax.set_ylabel('사망자 수')
            fig.tight_layout()
            st.pyplot(fig)
        if show_region:
            st.subheader("🗺 시도별 코로나 발생수 지도 산점도")
            st.write("시도별 누적 확진자 수를 기반으로 한반도 지도 위에 산점도로 표현한 그래프입니다.")
            st.write("점의 크기와 색이 누적 확진자 수에 비례합니다.")

            file_name = "누적.csv"
            try:
                df_region = pd.read_csv(file_name, encoding="utf-8-sig")
            except FileNotFoundError:
                st.error("'누적.csv' 파일을 찾을 수 없습니다.")
                st.stop()
            except Exception:
                df_region = pd.read_csv(file_name, encoding="cp437")

            # 전처리
            df_region.columns = df_region.columns.str.strip()

            # 첫 행이 '계' 합계 행이면 제거
            if "계" in df_region.iloc[0].values:
                df_region = df_region.drop(index=0).reset_index(drop=True)

            df_region = df_region.rename(columns={"시도명": "구분"})

            cols_to_numeric = ["누적확진자(명)"]
            for col in cols_to_numeric:
                df_region[col] = df_region[col].astype(str).str.replace(",", "", regex=False)
                df_region[col] = df_region[col].replace("-", "0")
            df_region[cols_to_numeric] = df_region[cols_to_numeric].apply(pd.to_numeric)

            # 검역 제외
            df_region = df_region[df_region["구분"] != "검역"].copy()

            # 시도 중심 좌표 (대략값)
            region_coords = {
                "서울": (37.5665, 126.9780),
                "부산": (35.1796, 129.0756),
                "대구": (35.8714, 128.6014),
                "인천": (37.4563, 126.7052),
                "광주": (35.1595, 126.8526),
                "대전": (36.3504, 127.3845),
                "울산": (35.5384, 129.3114),
                "세종": (36.4800, 127.2890),
                "경기": (37.4138, 127.5183),
                "강원": (37.8228, 128.1555),
                "충북": (36.6357, 127.4913),
                "충남": (36.5184, 126.8000),
                "전북": (35.7175, 127.1530),
                "전남": (34.8679, 126.9910),
                "경북": (36.4919, 128.8889),
                "경남": (35.4606, 128.2132),
                "제주": (33.4996, 126.5312),
            }

            df_region["lat"] = df_region["구분"].map(lambda x: region_coords.get(x, (None, None))[0])
            df_region["lon"] = df_region["구분"].map(lambda x: region_coords.get(x, (None, None))[1])

            # 좌표가 없는 행 제거
            df_region = df_region.dropna(subset=["lat", "lon"])

            # 산점도 지도 생성
            fig_region = px.scatter_mapbox(
                df_region,
                lat="lat",
                lon="lon",
                size="누적확진자(명)",
                color="누적확진자(명)",
                hover_name="구분",
                hover_data={"누적확진자(명)": ":,"},
                size_max=40,
                zoom=5.8,
                mapbox_style="carto-positron",
                color_continuous_scale="Reds",
            )

            fig_region.update_layout(
                margin=dict(l=0, r=0, t=0, b=0),
                coloraxis_colorbar=dict(title="누적 확진자(명)"),
            )

            st.plotly_chart(fig_region, use_container_width=True)


    if __name__ == "__main__":
        main()
