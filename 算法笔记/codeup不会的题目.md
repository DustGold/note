## 3.5  C

```c++
#include<cstdio>
int main(){
	char a[31];
	int len,temp,num[30],tar[200],tari,i,c;
	while(scanf("%s",a)!=-1){
		tari=0,i=0;
		for(len=0;a[len];len++){
			num[len]=a[len]-'0';
		}
		while(i<len){
			c=0;
			tar[tari++]=num[len-1]%2;
			for(int j=i;j<len;j++){
				temp=num[j];
				num[j]=(num[j]+c)/2;
				if(temp%2==1) c=10;
				else c=0;
			}
			if(num[i]==0) i++;
		}
		for(i=tari-1;i>=0;i--)
			printf("%d",tar[i]);
		printf("\n");
	}
}
```



 