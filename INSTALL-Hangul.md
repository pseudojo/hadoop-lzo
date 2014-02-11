Hadoop-LZO Install (Hangul Doc.)
==========


### 설치 방법 출처
- http://upepo.tistory.com/169
- http://opentsdb.net/setup-hbase.html

# 
#### 1. LZO 설치

##### 다운로드
	http://www.oberhumer.com/opensource/lzo/
    
##### 설치
	./configure --enable-shared
	make
	make install

##### 설치 확인
	ldconfig -v | grep liblzo

	콘솔 창에 다음과 유사한 내용이 표시되면 설치가 완료된 것이다.
		- liblzo2.so.2	->	liblzo2.so.2.0.0


# 
#### 2. 필요한 도구(Tools) 설치

##### Git, Ant 설치
	aptitude install git ant

# 
#### 3. 컴파일 및 배포

##### Github에서 복제
	git clone https://github.com/pseudojo/hadoop-lzo.git
    
##### Native 라이브러리로 컴파일 진행
	cd hadoop-lzo
    
    # 32비트의 경우
    CFLAGS=-m32 CXXFLAGS=-m32 ant compile-native tar
    
    # 64비트의 경우
    CFLAGS=-m64 CXXFLAGS=-m64 ant compile-native tar

	HBASE_DIR=path/to/hbase
    mkdir -p $HBASE_DIR/lib/native
    cp build/hadoop-lzo-0.4.15/hadoop-lzo-0.4.15.jar $HBASE_DIR/lib
    cp -a build/hadoop-lzo-0.4.15/lib/native/* $HBASE_DIR/lib/native
	
    
##### 추가 사항
	# 필요한 경우 .bash_profile 또는 .bashrc 와 같은 PATH 설정 파일에 다음의 내용을 추가한다.

	JAVA_LIBRARY_PATH=$JAVA_LIBRRAY_PATH:$HADOOP_HOME/lib/
	JAVA_LIBRARY_PATH=$JAVA_LIBRRAY_PATH:$HADOOP_HOME/lib/native/Linux-amd64-64/
    export JAVA_LIBRARY_PATH


# 
#### 4. Hadoop 설정 파일 수정

	# core-site.xml
	# LZO Compression도 기본 코덱으로 설정 
        <property>
			<name>io.compression.codecs</name>
        	<value>org.apache.hadoop.io.compress.GzipCodec,
				org.apache.hadoop.io.compress.DefaultCodec,
				org.apache.hadoop.io.compress.BZip2Codec,
				com.hadoop.compression.lzo.LzoCodec
            </value>
        </property>

        <property>
			<name>io.compression.codec.lzo.class</name>
			<value>com.hadoop.compression.lzo.LzoCodec</value>
        </property>
        
    
    # mapred-site.xml
	# Map이 끝나고 압축해서 Reduce로 전달
        <property>
			<name>mapred.compress.map.output</name>
			<value>true</value>
        </property>
        <property>
			<name>mapred.map.output.compression.codec</name>
			<value>com.hadoop.compression.lzo.LzoCodec</value>
        </property>

	# Reduce 끝나고 최종 결과 압축
        <property>
			<name>mapred.output.compress</name>
			<value>true</value>
        </property>
        <property>
			<name>mapred.output.compression.codec</name>
			<value>com.hadoop.compression.lzo.LzoCodec</value>
        </property>
        
        